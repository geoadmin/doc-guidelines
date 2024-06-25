# Terraform Best Practices and Guidelines

General terraform guidelines and best practices

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Code Organization](#code-organization)
- [State Management](#state-management)
  - [Remote State](#remote-state)
    - [State key](#state-key)
  - [State Locking](#state-locking)
  - [State migration](#state-migration)
- [Module](#module)
  - [Module Guidelines](#module-guidelines)
  - [Reusable/child Module Usage](#reusablechild-module-usage)
- [Development Guidelines](#development-guidelines)
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
|           |- ENVIRONMENT    # resource pro environment
```

Then it should mirror the hierarchical structure of the infrastructure (see [PP BGDI Infra Architecture](https://ltwiki.adr.admin.ch:8443/display/PB/Runtime+Infrastructure+Stack)). Note that each terraform module (project) should be kept small.

## State Management

### Remote State

We use remote state management using AWS S3 backend

- Encrypt the state file at rest (see [Terraform Documentation](https://developer.hashicorp.com/terraform/language/settings/backends/s3#encrypt)).
- Put the state backend definition in `terraform.tf` file
- For AWS ressources we use one state bucket per AWS Account
- All terraform code relating to a remote state HAVE TO BE in the same git repository. However you can have several state buckets in the same git repository, but they need to be clearly separated as in code organization above.
- Cross account state bucket access IS NOT ALLOWED. This allow a simpler terraform state bucket access management and also improve security. This means that we cannot get remote state from another account.
  - Improve security as one developer might have access to one account but the other, which means developer is only allow to use terraform on the account it has access.
  - Simplify state key migration, as we have to check for remote state only in one github repository
- Inside the same account (AWS, github organization, ...) we can use remote state to access other root module data to avoid hardcoding

#### State key

We use the terraform module definition file path as state key in S3 prefixed with a version string `v1`; e.g. `v1/infra-terraform-bgdi/systems-prod/map.terraform.tf`

The version prefix is to avoid any risk of key collision in the future in case of code structure re-organization.

*NOTE: We used to use an UUID as state keys, which has the advantage that we did not have to change the state when reorganizing resources. However it has the downside that UUID key were very hard to code review and error prone due to copy paste mistake not being easily identifiable.*

### State Locking

Enable state locking to prevent concurrent operations from corrupting your state.

- Use AWS DynamoDB to manage state locks.

### State migration

To migrate the state we use the `terraform init -migrate-state` command. After migrating the state we need to manually clean up the old state by removing the S3 file and deleting the dynamoDB entry

Here is an example of migrate state procedure

1. Run the `terraform init` BEFORE changing the state key on the terraform backend file :warning:
2. Update the the state key in the terraform backend file
3. Run `terraform init -migrate-state`
4. Clean the old state in S3 and DynamoDB

```bash
OLD_KEY=<old-state-key>

aws --profile AWS_ACCOUNT_PROFILE s3 rm s3://STATE_BUCKET/${OLD_KEY}

aws --profile AWS_ACCOUNT_PROFILE dynamodb delete-item --table-name TABLE_NAME --key '{"LockID": {"S": "STATE_BUCKET/'${OLD_KEY}'-md5"}}'
```

## Module

*Modules are containers for multiple resources that are used together.* See [Terraform Modules](https://developer.hashicorp.com/terraform/language/modules)

### Module Guidelines

- Root module must contain the terraform backend definition
- Avoid using magic number like AWS account id in your code but use variables, locals or global variables defined in reusable module instead
- Avoid hard dependencies between modules whenever possible. By hard dependencies, I mean using remote state for a value when the use of the value in the resource doesn't need to exists, for example for AWS policies and role you might be tempted to get the role ARN or resource ARN from a remote state to create a new role or policy, but this would add a hard dependency on the terraform module where one module needs to be applied before the other, while on the AWS resource you don't have any hard dependencies, the ARN resource in a policy doesn't need to exists to create the policy.
- Pay attention to circlar dependencies between modules, when the resource already exists and are imported to terraform, you don't have circular dependencies but by re-creating from scratch it might be the case !

### Reusable/child Module Usage

Use modules to encapsulate and reuse configurations. This promotes DRY (Don't Repeat Yourself) principles and makes your configurations more manageable.

- Create modules for common resources.
- Use versioned modules from the Terraform Registry when possible.
- Use modules to share global variales between terraform modules
- Pay attention to the drawback of reusable modules, were changing it might impact lots of other root modules

## Development Guidelines

- :warning: ALWAYS change resources using terraform (no manual changes via UI, e.g. AWS web console) :warning:
- :warning: NEVER EVER apply changes before opening a Pull Request :warning:
- `master` branch SHOULD reflect the current infrastructure state, which means PR that are applied should be merged as soon as they have been reviewed. DON'T keep applied PR open for more that a few days.
- Keep track of applied changes in the PR using the checkbox
  - [ ] changes applied
  
  Modify/add/remove checkbox if needed (e.g. one checkbox per module/staging, etc)
  
## Code formatting

All terraform code should have the same formatting, for this we use the standard `terraform fmt` tool to format the code.
