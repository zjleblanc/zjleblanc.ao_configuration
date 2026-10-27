# zjleblanc.ao_configuration.ao_tools

## Description

An Ansible Role to enable/disable and update metadata for Tools (`/api/v1/tools`) on
Automation Orchestrator. Tools are **auto-discovered from enabled Integrations** -- the
API has no create or delete endpoint for them, so this role is update-only. If a named
tool doesn't exist yet (e.g. because its owning integration hasn't been created/enabled
yet), the role fails with a clear message rather than silently skipping it.

## Requirements

No external collection dependencies -- this role uses only `ansible.builtin` modules.

## Variables

| Variable Name                 | Default Value | Required | Description                                                                 |
| :------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_hostname`                      | `""`                | yes      | URL to the Automation Orchestrator server.                                  |
| `ao_validate_certs`                | `true`               | no       | Whether or not to validate the AO server's SSL certificate.                 |
| `ao_access_token`                   | `""`                | yes      | Bearer token. Auto-managed by the `ao_auth` role dependency.                 |
| `ao_request_timeout`               | `10`                 | no       | Specify the timeout in seconds Ansible should use in requests to AO.        |
| `ao_configuration_collect_logs`    | `false`              | no       | Collect async results and continue for all failed tasks instead of failing on the first error. |
| `ao_configuration_register`         | `""`                | no       | Variable to register the values of all `ao_configuration` tasks.            |
| `ao_tools`                            | `see below`          | yes      | Data structure describing the tools to update. Described below.             |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
tools (up to `ao_configuration_tools_lookup_limit`) and matches on `name` client-side.

| Variable Name                           | Default Value | Required | Description                                                                 |
| :------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_tools_lookup_limit`            | `200`              | no       | Maximum number of existing tools to fetch for name matching.                |
| `ao_configuration_tools_list_key`                | `items`            | no       | Key in the list response containing the array of tools. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                      | Default Value | Required | Description                                                                 |
| :-------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_tools_secure_logging`      | `false`            | no       | Whether or not to include the tool role tasks in the log.                   |
| `ao_configuration_secure_logging`             | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                               | Default Value                       | Required | Description                                                                 |
| :----------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                   | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_tools_async_retries`              | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                     | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_tools_async_delay`                | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                      | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_tools_loop_delay`                 | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                        | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Tool Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `name`               | yes      | Name of the (already auto-discovered) tool.                                  |
| `enabled`            | no       | Whether the tool is enabled. Defaults to `true`.                             |
| `description`         | no       | Override the tool's description. Defaults to leaving it unchanged.          |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_tools:
  - name: slack_send_message
    enabled: true
  - name: servicenow_create_incident
    enabled: false
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_tools, when: ao_tools is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zack LeBlanc](https://github.com/zjleblanc)
