# zjleblanc.ao_configuration.ao_role_assignments

## Description

An Ansible Role to create/remove Role Assignments (`/api/v1/role_assignments`) on
Automation Orchestrator. A role assignment binds a role to a subject (a user, a group, or
a service account), optionally scoped to a single project.

Role assignments are immutable -- the API only supports creating and deleting them, not
updating them in place. This role mirrors that: it will never `PATCH` an assignment, only
`POST` (create) or `DELETE` (remove) based on whether a matching assignment already
exists.

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
| `ao_role_assignments`                 | `see below`          | yes      | Data structure describing your role assignments. Described below.           |

### Lookup Variables

Since the AO list endpoints do not support filtering, this role lists existing role
assignments (up to `ao_configuration_role_assignments_lookup_limit`) and matches on the
`role_name`/`subject_type`/`subject_name`/`project_id` tuple client-side to decide whether
an assignment already exists.

| Variable Name                                      | Default Value | Required | Description                                                                 |
| :-------------------------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_role_assignments_lookup_limit`            | `100`              | no       | Maximum number of existing role assignments to fetch for matching.          |
| `ao_configuration_role_assignments_list_key`                | `items`            | no       | Key in the list response containing the array of role assignments. Adjust if your AO deployment's response envelope differs. |

### Secure Logging Variables

| Variable Name                                   | Default Value | Required | Description                                                                 |
| :------------------------------------------------------ | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_role_assignments_secure_logging`           | `false`            | no       | Whether or not to include the role assignment role tasks in the log.        |
| `ao_configuration_secure_logging`                              | `false`            | no       | Shared toggle across the whole collection; see above.                       |

### Asynchronous Retry Variables

| Variable Name                                           | Default Value                       | Required | Description                                                                 |
| :------------------------------------------------------------- | :-------------------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_async_retries`                                  | 50                                     | no       | This variable sets the number of retries to attempt for the role globally.   |
| `ao_configuration_role_assignments_async_retries`                   | `{{ ao_configuration_async_retries }}`  | no       | This variable sets the number of retries to attempt for this role.           |
| `ao_configuration_async_delay`                                    | 1                                       | no       | This sets the delay between retries for the role globally.                   |
| `ao_configuration_role_assignments_async_delay`                     | `ao_configuration_async_delay`          | no       | This sets the delay between retries for this role.                           |
| `ao_configuration_loop_delay`                                     | 0                                       | no       | This sets the pause between each item in the loop for the roles globally.    |
| `ao_configuration_role_assignments_loop_delay`                      | `ao_configuration_loop_delay`           | no       | This sets the pause between each item in the loop for this role.             |
| `ao_configuration_async_dir`                                       | `null`                                   | no       | Sets the directory to write the results file for async tasks.               |

## Data Structure

### Role Assignment Variables

| Variable Name   | Required | Description                                                                 |
| :----------------- | :------: | :----------------------------------------------------------------------------- |
| `role_name`          | yes      | Name of the role to assign.                                                   |
| `subject_type`        | yes      | One of `user`, `group`, or `service_account`.                                |
| `subject_name`        | yes      | Username, group name, or service account name, depending on `subject_type`.  |
| `project_id`          | no       | Scope the assignment to a single project. Omit for a global/system-wide assignment. |
| `state`              | no       | Desired state of the resource: `present` or `absent`. Default `present`.    |
| `register`           | no       | Variable to set based on the result of the object creation/modification.    |

### Yaml Example

```yaml
---
ao_role_assignments:
  - role_name: Workflow Operator
    subject_type: group
    subject_name: Platform Admins
    project_id: "{{ project_id }}"
  - role_name: Admin
    subject_type: user
    subject_name: jdoe
```

## Playbook Example

```yaml
---
- name: Playbook to configure Automation Orchestrator post installation
  hosts: localhost
  connection: local
  roles:
    - { role: zjleblanc.ao_configuration.ao_role_assignments, when: ao_role_assignments is defined }
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
