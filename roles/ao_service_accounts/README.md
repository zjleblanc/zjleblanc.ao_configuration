# zjleblanc.ao_configuration.ao_service_accounts

## Description

An Ansible Role to create/update/remove/enable/disable Service Accounts
(`/api/v1/service_accounts`) on Automation Orchestrator. Service accounts are used for
`client_credentials` OAuth2 authentication (see the `ao_auth` role).

Setting `state: enabled` or `state: disabled` on an existing service account routes to
the dedicated `/api/v1/service_accounts/{id}/enable` and `/.../disable` sub-endpoints
rather than the generic `PATCH`.

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
| `ao_service_accounts`                | `see below`          | yes      | Data structure describing your service accounts. Described below.           |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
service accounts (up to `ao_configuration_service_accounts_lookup_limit`) and matches on
`name` client-side to decide whether to create, update, enable, or disable.

| Variable Name                                      | Default Value | Required | Description                                                                 |
| :------------------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_service_accounts_lookup_limit`           | `100`              | no       | Maximum number of existing service accounts to fetch for name matching.     |
| `ao_configuration_service_accounts_list_key`               | `items`            | no       | Key in the list response containing the array of service accounts. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                                    | Default Value | Required | Description                                                                 |
| :----------------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_service_accounts_secure_logging`          | `true`             | no       | Whether or not to include the sensitive service account role tasks in the log. |
| `ao_configuration_secure_logging`                            | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                            | Default Value                       | Required | Description                                                                 |
| :------------------------------------------------------------ | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                                 | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_service_accounts_async_retries`                 | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                                   | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_service_accounts_async_delay`                   | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                                    | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_service_accounts_loop_delay`                    | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                                      | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Service Account Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `name`               | yes      | Name of the service account.                                                  |
| `description`        | no       | Description of the service account.                                          |
| `project_id`          | no       | Project this service account belongs to.                                     |
| `state`              | no       | Desired state: `present`, `absent`, `enabled`, or `disabled`. Default `present`. |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_service_accounts:
  - name: ci-pipeline
    description: Service account for CI pipeline automation
    project_id: "{{ project_id }}"
    state: enabled
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_service_accounts, when: ao_service_accounts is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
