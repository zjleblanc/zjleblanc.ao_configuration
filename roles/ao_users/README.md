# zjleblanc.ao_configuration.ao_users

## Description

An Ansible Role to create/update/remove local Users (`/api/v1/users`) on Automation
Orchestrator.

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
| `ao_users`                            | `see below`          | yes      | Data structure describing your users. Described below. Alias: `users`.      |

### Lookup Variables

Since the AO list endpoints do not support filtering by username, this role lists
existing users (up to `ao_configuration_users_lookup_limit`) and matches on `username`
client-side to decide whether to create or update.

| Variable Name                           | Default Value | Required | Description                                                                 |
| :------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_users_lookup_limit`           | `100`              | no       | Maximum number of existing users to fetch for username matching.            |
| `ao_configuration_users_list_key`               | `items`            | no       | Key in the list response containing the array of users. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                      | Default Value | Required | Description                                                                 |
| :-------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_users_secure_logging`      | `true`             | no       | Whether or not to include the sensitive user role tasks in the log (passwords). |
| `ao_configuration_secure_logging`             | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                               | Default Value                       | Required | Description                                                                 |
| :----------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                   | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_users_async_retries`              | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                     | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_users_async_delay`                | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                      | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_users_loop_delay`                 | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                        | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### User Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `username`           | yes      | Username of the local user.                                                  |
| `first_name`          | no       | First name.                                                                    |
| `last_name`           | no       | Last name.                                                                     |
| `email`               | no       | Email address.                                                                 |
| `password`            | no       | Password. Since the API never returns password values back, this role always re-applies `password` on update when it is set rather than diffing it. |
| `is_enabled`          | no       | Whether the user is enabled. Defaults to `true`.                             |
| `group_names`         | no       | List of group names this user belongs to.                                   |
| `state`               | no       | Desired state of the resource. Default `present`.                           |
| `register`            | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_users:
  - username: jdoe
    first_name: Jane
    last_name: Doe
    email: jdoe@example.com
    password: "{{ vault_jdoe_password }}"
    is_enabled: true
    group_names: []
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_users, when: ao_users is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
