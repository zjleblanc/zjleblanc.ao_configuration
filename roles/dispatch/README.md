# zjleblanc.ao_configuration.dispatch

## Description

An Ansible role which conditionally calls every resource role in the `zjleblanc.ao_configuration`
collection, based on which top-level variables are defined. This is the role that playbooks
should call in order to apply an entire configuration in one run.

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  gather_facts: false
  pre_tasks:
    - name: Include vars from configs directory
      ansible.builtin.include_vars:
        dir: ./configs
        extensions: [yml]
  roles:
    - zjleblanc.ao_configuration.dispatch
```

Each entry in `ao_configuration_dispatcher_roles` is only run `when: vars[__role.var] is
defined`, so you only need to define the variables for the resources you actually want to
manage (e.g. only set `ao_projects` and `ao_credentials` if that's all you need).

## Variables

| Variable Name                           | Default Value                                                     | Required | Description                                                                 |
| :----------------------------------------- | :-------------------------------------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_dispatcher_roles`          | [See `defaults/main.yml`](defaults/main.yml)                        | no       | The ordered list of `{role, var, tags}` entries the dispatcher iterates over. |
| `ao_configuration_dispatcher_exclude_roles`  | `[]`                                                                  | no       | List of role names to skip entirely, even if their variable is defined.      |
| `dispatch_include_wildcard_vars`             | `false`                                                                | no       | When `true`, merges wildcard-suffixed variables (see below) before dispatching. |
| `ao_configuration_collect_logs`              | `false`                                                                | no       | When `true`, collect every resource role failure instead of failing on the first one. |
| `ao_configuration_role_errors`               | n/a (populated at runtime)                                           | no       | Dictionary of collected failures, keyed by `<role>_errors` (only populated when `ao_configuration_collect_logs` is `true`). |

### Wildcard Variables

Setting `dispatch_include_wildcard_vars: true` allows you to split any resource variable
across multiple suffixed variables -- for example, to separate configuration by
environment or by team -- and have them automatically combined into the base variable
before dispatching:

```yaml
---
dispatch_include_wildcard_vars: true

ao_credentials_team_a:
  - name: team-a-credential
    credential_type: api_key
    inputs:
      api_key: "{{ vault_team_a_api_key }}"

ao_credentials_team_b:
  - name: team-b-credential
    credential_type: api_key
    inputs:
      api_key: "{{ vault_team_b_api_key }}"
```

This is equivalent to defining a single combined `ao_credentials` list containing both
entries. Any variable matching `^(<resource>)_.*` (excluding the role-tuning suffixes
`_secure_logging`, `_async_retries`, `_async_delay`, `_loop_delay`, `_enforce_defaults`) is
merged into its base resource variable.

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zack LeBlanc](https://github.com/zjleblanc)
