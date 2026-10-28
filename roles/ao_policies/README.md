# zjleblanc.ao_configuration.ao_policies

## Description

An Ansible Role to create/update/remove Policies (`/api/v1/policies`) on Automation
Orchestrator. Policies express guardrail/governance rules evaluated against workflow
execution requests.

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
| `ao_policies`                        | `see below`          | yes      | Data structure describing your policies. Described below.                   |

### Lookup Variables

Since the AO list endpoints do not support filtering by name, this role lists existing
policies (up to `ao_configuration_policies_lookup_limit`) and matches on `name`
client-side to decide whether to create or update.

| Variable Name                              | Default Value | Required | Description                                                                 |
| :----------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_policies_lookup_limit`             | `100`              | no       | Maximum number of existing policies to fetch for name matching.             |
| `ao_configuration_policies_list_key`                 | `items`            | no       | Key in the list response containing the array of policies. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                          | Default Value | Required | Description                                                                 |
| :------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_policies_secure_logging`       | `false`            | no       | Whether or not to include the policy role tasks in the log.                 |
| `ao_configuration_secure_logging`                 | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                   | Default Value                       | Required | Description                                                                 |
| :---------------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                         | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_policies_async_retries`                 | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                           | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_policies_async_delay`                   | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                            | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_policies_loop_delay`                    | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                              | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Policy Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `name`               | yes      | Name of the policy.                                                           |
| `description`        | no       | Description of the policy.                                                   |
| `rules`              | no       | List of rule definitions evaluated by this policy.                          |
| `enabled`            | no       | Whether the policy is enabled. Defaults to `true`.                           |
| `state`              | no       | Desired state of the resource. Default `present`.                           |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_policies:
  - name: Require Approval for Prod Workflows
    description: Guardrail requiring manual approval for prod-tagged workflows
    enabled: true
    rules:
      - condition: "labels.environment == 'prod'"
        action: require_approval
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_policies, when: ao_policies is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
