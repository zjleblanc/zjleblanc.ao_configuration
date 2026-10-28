# zjleblanc.ao_configuration.ao_settings

## Description

An Ansible Role to update runtime settings (`/api/v1/settings`) on Automation Orchestrator.
Settings are a keyed singleton resource -- they always exist, so this role only ever
updates (`PATCH`) them; it never creates or deletes a setting.

AO settings use optimistic locking via `expected_version`. If you don't supply
`expected_version` yourself, this role looks up the setting's current version
immediately beforehand and uses that -- which is safe for sequential runs but can race if
something else updates the same setting concurrently with your playbook run.

## Requirements

No external collection dependencies -- this role uses only `ansible.builtin` modules.

## Variables

| Variable Name                           | Default Value | Required | Description                                                                 |
| :----------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_hostname`                               | `""`               | yes      | URL to the Automation Orchestrator server.                                   |
| `ao_validate_certs`                         | `true`             | no       | Whether or not to validate the AO server's SSL certificate.                  |
| `ao_access_token`                            | `""`               | yes      | Bearer token. Auto-managed by the `ao_auth` role dependency.                  |
| `ao_request_timeout`                        | `10`               | no       | Specify the timeout in seconds Ansible should use in requests to AO.         |
| `ao_configuration_collect_logs`             | `false`            | no       | Collect async results and continue for all failed tasks instead of failing on the first error. |
| `ao_configuration_register`                  | `""`               | no       | Variable to register the values of all `ao_configuration` tasks.             |
| `ao_settings`                                | `see below`        | yes      | Data structure describing the settings to update. Described below.           |

### Asynchronous Retry Variables

| Variable Name                               | Default Value                       | Required | Description                                                                 |
| :---------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                 | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_settings_async_retries`         | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                   | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_settings_async_delay`           | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                    | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_settings_loop_delay`            | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                      | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Setting Variables

| Variable Name        | Required | Description                                                                 |
| :---------------------- | :------: | :----------------------------------------------------------------------------- |
| `key`                    | yes      | Dot-namespaced setting key (e.g. `files.max_upload_size_mb`).                |
| `value`                  | yes      | The desired value for this setting.                                        |
| `expected_version`       | no       | Version for optimistic-locking. If omitted, this role looks it up for you.  |

### Yaml Example

```yaml
---
ao_settings:
  - key: files.max_upload_size_mb
    value: 10
  - key: integrations.default_timeout_seconds
    value: 30
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_settings, when: ao_settings is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
