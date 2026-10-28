# zjleblanc.ao_configuration.ao_credentials

## Description

An Ansible Role to create/update/remove Credentials (`/api/v1/credentials`) on Automation
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
| `ao_credentials`                     | `see below`          | yes      | Data structure describing your credentials. Described below. Alias: `credentials`. |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
credentials (up to `ao_configuration_credentials_lookup_limit`, metadata only -- no
secrets) and matches on `name` client-side to decide whether to create or update.

| Variable Name                                | Default Value | Required | Description                                                                 |
| :----------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_credentials_lookup_limit`          | `100`              | no       | Maximum number of existing credentials to fetch for name matching.          |
| `ao_configuration_credentials_list_key`              | `items`            | no       | Key in the list response containing the array of credentials. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                              | Default Value | Required | Description                                                                 |
| :------------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_credentials_secure_logging`          | `true`             | no       | Whether or not to include the sensitive credential role tasks in the log.   |
| `ao_configuration_secure_logging`                       | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                      | Default Value                       | Required | Description                                                                 |
| :------------------------------------------------------ | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                           | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_credentials_async_retries`               | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                             | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_credentials_async_delay`                 | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                              | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_credentials_loop_delay`                  | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                                | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Credential Variables

| Variable Name          | Required | Description                                                                 |
| :------------------------ | :------: | :----------------------------------------------------------------------------- |
| `name`                      | yes      | Name of the credential.                                                      |
| `description`               | no       | Description of the credential.                                               |
| `credential_type_id`         | yes      | ID of the credential type.                                                   |
| `project_id`                 | no       | Project this credential belongs to.                                         |
| `inputs`                      | no       | Credential inputs (secret values), keyed per the credential type's schema. Since the API never returns secret values back, this role always re-applies `inputs` on update rather than diffing them. |
| `labels`                      | no       | Arbitrary key/value labels.                                                   |
| `state`                       | no       | Desired state of the resource. Default `present`.                           |
| `register`                    | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_credentials:
  - name: My API Key
    description: Third-party API credential
    credential_type_id: "{{ credential_type_id }}"
    project_id: "{{ project_id }}"
    inputs:
      api_key: "{{ vault_my_api_key }}"
    labels: {}
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_credentials, when: ao_credentials is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
