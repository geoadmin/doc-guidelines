# Ansible guidelines

- [Naming convention](#naming-convention)
- [Linting](#linting)
- [Formatting](#formatting)
- [Role vs Tasks](#role-vs-tasks)
- [Repository Structure](#repository-structure)
  - [Collection](#collection)

## Naming convention

File, directory and variable names MUST be `snake_case`. Only use `yaml` files and use `*.yml` extension.

## Linting

Code should pass linting

```bash
ansible-lint
```

## Formatting

Code should be formatted as follow:

- Use space instead of tab
- Use 2 spaces indentation
- YAML files should always start with `---` and end with a blank line

    ```yaml
    ---
    # This is an example of correct yaml file formatted
    my_yaml:
      content: hello world
    
    ```

You should always use the formatting tool of ansible-lint

```bash
ansible-lint --write all
```

## Role vs Tasks

For reusable code prefer Roles over Tasks. If some code could reused than encapsulate it in a role,
otherwise simply add it to your playbook.

## Repository Structure

Repository should be prefixed by; `infra-ansible-`

And they should follow this structure

```text
├── files
│   └── some-file.txt
├── group_vars
│   └── group_name.yml
├── hosts_vars
│   └── host_name.yml
├── roles
│   └── my_role
│       ├── handlers
│       │   └── main.yml
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   └── some-file.txt
│       ├── tasks
│       │   └── main.yml
│       ├── vars
│       │   └── main.yml
│       └── README.md
├── vars
│   └── main.yml
├── playbook_1.yml
├── playbook_2.yml
└── inventory.yml
```

### Collection

For collection we follow the ansible standard folder structure

```text
├── roles
│   └── bgdi_toolchains
│       ├── handlers
│       ├── defaults
│       ├── files
│       ├── tasks
│       │   └── main.yml
│       ├── vars
│       └── README.md
├── playbooks
|   ├── files
|   ├── vars
|   ├── templates
│   └── my-playbook.yml
├── galaxy.yml
└── README.md
```
