# zjleblanc.ao_configuration.ao_projects

## Description

An Ansible Role to create/update/remove Projects (`/api/v1/projects`) on Automation
Orchestrator. Projects are the top-level organizational container that credentials,
workflows, integrations, and service accounts belong to.

## Requirements

No external collection dependencies -- this role uses only `ansible.builtin` modules.

## Variables

| Variable Name                 | Default Value | Required | Description                                                                 |
| :------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_state`                         | `"present"`         | no       | The state all objects will take unless overridden by object default.        |
| `ao_hostname`                      | `""`                | yes      | URL to the Automation Orchestrator server.                                  |
| `ao_validate_certs`                | `true`               | no       | Whether or not to validate the AO server's SSL certificate.                 |
| `ao_access_token`                   | `""`                | yes      | Bearer token. Auto-managed by the `ao_auth` role dependency.                 |
| `ao_request_timeout`               | `10`                 | no       | Specify the timeout in seconds Ansible should use in requests to AO.        |
| `ao_configuration_collect_logs`    | `false`              | no       | Collect async results and continue for all failed tasks instead of failing on the first error. |
| `ao_configuration_register`         | `""`                | no       | Variable to register the values of all `ao_configuration` tasks.            |
| `ao_projects`                        | `see below`          | yes      | Data structure describing your projects. Described below. Alias: `projects`. |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
projects (up to `ao_configuration_projects_lookup_limit`) and matches on `name`
client-side to decide whether to create or update.

| Variable Name                               | Default Value | Required | Description                                                                 |
| :---------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_projects_lookup_limit`           | `100`              | no       | Maximum number of existing projects to fetch for name matching.             |
| `ao_configuration_projects_list_key`               | `items`            | no       | Key in the list response containing the array of projects. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                           | Default Value | Required | Description                                                                 |
| :------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_projects_secure_logging`       | `false`            | no       | Whether or not to include the project role tasks in the log.                |
| `ao_configuration_secure_logging`                 | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                     | Default Value                       | Required | Description                                                                 |
| :------------------------------------------------------ | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                           | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_projects_async_retries`                   | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                             | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_projects_async_delay`                     | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                              | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_projects_loop_delay`                      | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                                | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Project Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `name`               | yes      | Name of the project.                                                          |
| `description`        | no       | Description of the project.                                                  |
| `labels`             | no       | Arbitrary key/value labels.                                                   |
| `state`              | no       | Desired state of the resource. Default `present`.                           |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_projects:
  - name: My Project
    description: Project description
    labels: {}
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_projects, when: ao_projects is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
