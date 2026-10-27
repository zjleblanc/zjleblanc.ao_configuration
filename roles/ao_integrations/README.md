# zjleblanc.ao_configuration.ao_integrations

## Description

An Ansible Role to create/update/remove Integrations (`/api/v1/integrations`) on
Automation Orchestrator. Integrations connect AO to external systems (e.g. Slack,
ServiceNow, a source control provider) and are typically backed by a `ao_credentials`
entry. Tools (`ao_tools`) are auto-discovered from enabled integrations.

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
| `ao_integrations`                    | `see below`          | yes      | Data structure describing your integrations. Described below.               |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
integrations (up to `ao_configuration_integrations_lookup_limit`) and matches on `name`
client-side to decide whether to create or update.

| Variable Name                                 | Default Value | Required | Description                                                                 |
| :------------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_integrations_lookup_limit`          | `100`              | no       | Maximum number of existing integrations to fetch for name matching.         |
| `ao_configuration_integrations_list_key`              | `items`            | no       | Key in the list response containing the array of integrations. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                           | Default Value | Required | Description                                                                 |
| :-------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_integrations_secure_logging`    | `true`             | no       | Whether or not to include the sensitive integration role tasks in the log (configuration may reference secrets). |
| `ao_configuration_secure_logging`                  | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                       | Default Value                       | Required | Description                                                                 |
| :--------------------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                              | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_integrations_async_retries`                   | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                                 | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_integrations_async_delay`                      | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                                  | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_integrations_loop_delay`                       | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                                    | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Integration Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `name`               | yes      | Name of the integration.                                                      |
| `description`        | no       | Description of the integration.                                              |
| `integration_type`    | yes      | Type of the integration (e.g. `slack`, `servicenow`, `github`).              |
| `project_id`          | no       | Project this integration belongs to.                                         |
| `credential_id`        | no       | Credential used to authenticate this integration.                           |
| `configuration`        | no       | Integration-type-specific configuration dict.                               |
| `enabled`             | no       | Whether the integration is enabled. Defaults to `true`.                      |
| `state`              | no       | Desired state of the resource. Default `present`.                           |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_integrations:
  - name: Team Slack
    integration_type: slack
    project_id: "{{ project_id }}"
    credential_id: "{{ slack_credential_id }}"
    configuration:
      default_channel: "#automation"
    enabled: true
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_integrations, when: ao_integrations is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zack LeBlanc](https://github.com/zjleblanc)
