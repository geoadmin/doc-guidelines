# Terraform Best Practices and Guidelines

General terraform guidelines and best practices

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Code Organization](#code-organization)
  - [Reusable Module Usage](#reusable-module-usage)
- [State Management](#state-management)
  - [Remote State](#remote-state)
    - [State key](#state-key)
  - [State Locking](#state-locking)
  - [State migration](#state-migration)
  - [Module](#module)
- [Code formatting](#code-formatting)

## Introduction

Terraform is a powerful tool for provisioning and managing infrastructure as code. To ensure maintainability, scalability, and security, it's important to follow best practices and guidelines when working with Terraform.

## Code Organization

The high level code organization should look like this

```text
|- modules/                   # terraform reusable modules for the whole repository
|- ORGANIZATIONS_OR_ACCOUNT   # e.g. AWS Account name
|           |- modules        # terraform reusable modules for this organization/account
|           |- shared         # shared resources
|           |- STAGING        # resource pro staging
```

Then it should mirror the hierarchical structure of the infrastructure. Note that each terraform module (project) should be kept small.

### Reusable Module Usage

Use modules to encapsulate and reuse configurations. This promotes DRY (Don't Repeat Yourself) principles and makes your configurations more manageable.

- Create modules for common resources.
- Use versioned modules from the Terraform Registry when possible.
- Use modules to share global variales between terraform modules

## State Management

### Remote State

We use remote state management using AWS S3 backend

- Encrypt the state file at rest.
- Put the state backend definition in `terraform.tf` file

#### State key

We use the terraform module definition file path as state key in S3 prefixed with a version string `v1`; e.g. `v1/infra-terraform-bgdi/systems-prod/map.terraform.tf`

The version prefix is to avoid any risk of key collision in future in case of code structure re-organization.

*NOTE: We used to use an UUID as state key, which has the advantage that we did not have to change the state when reorganizing resources. However it has the downside that UUID key were very hard to code review and error prone due to copy paste mistake not being easily identifiable.*

### State Locking

Enable state locking to prevent concurrent operations from corrupting your state.

- Use AWS DynamoDB to manage state locks.

### State migration

To migrate the state we use the `terraform init --migrate-state` command. After migrating the state we need to manually clean up the old state by removing the S3 file and deleting the dynamoDB entry

```bash
OLD_KEY=<old-state-key>

aws --profile AWS_ACCOUNT_PROFILE s3 rm s3://STATE_BUCKET/${OLD_KEY}

aws --profile AWS_ACCOUNT_PROFILE dynamodb delete-item --table-name TABLE_NAME --key '{"LockID": {"S": "STATE_BUCKET/'${OLD_KEY}'-md5"}}'
```

### Module

- For AWS ressources we use one state bucket per AWS Account, we don't allow cross account state bucket access. This allow a simpler terraform state bucket access management and also improve security.
- Inside the same AWS account we can use remote state to access other terraform module data to avoid hardcoding
- Avoid using magic number like AWS account id in your code but use variables, locals or global variables defined in reusable module instead

## Code formatting

All terraform code should have the same formatting, for this we use the standard `terraform fmt` tool to format the code.
