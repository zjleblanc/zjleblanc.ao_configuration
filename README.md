# zjleblanc.ao_configuration

This Ansible collection allows for easy interaction with the Automation Orchestrator (AO) API via Ansible roles, following the same resource-based role, dispatch, and wildcard variable patterns established by [infra.aap_configuration](https://github.com/redhat-cop/infra.aap_configuration).

## Description

[Automation Orchestrator (AO)](https://github.com/zjleblanc/autodotes-postman-collections/tree/main/postman/collections/AO) (a.k.a. Syntara Orchestration) is a distributed multi-agent orchestration API. This collection manages AO objects (projects, credentials, workflows, users, integrations, RBAC, and more) declaratively, via `/api/v1/...`.

Unlike `infra.aap_configuration`, which relies on certified `ansible.controller`/`ansible.platform` modules, AO has no certified Ansible modules. Every resource role in this collection therefore uses `ansible.builtin.uri` directly against the AO REST API, while preserving the exact same role layout, dispatch mechanics, and documentation conventions.

## Requirements

No external collection dependencies -- every role uses only `ansible.builtin` modules.

## Authentication

The AO API does not support a long-lived personal access token. Every resource role depends (via `meta/main.yml`) on the `ao_auth` role, which acquires and caches a short-lived JWT, re-authenticating automatically whenever it is missing or expired. Because Ansible deduplicates identical role dependencies within a play, this login happens once per run no matter how many resource roles execute.

Two authentication methods are supported -- set one pair of variables:

| Variable Name      | Required | Description                                                                 |
| :------------------ | :------: | :--------------------------------------------------------------------------- |
| `ao_client_id`       | no       | Service account client ID. Used with `ao_client_secret` for the OAuth2 `client_credentials` grant (`POST /api/v1/auth/token`). Preferred for automation. |
| `ao_client_secret`   | no       | Service account client secret.                                              |
| `ao_username`        | no       | Username for interactive login (`POST /api/v1/auth/login`). Used when `ao_client_id`/`ao_client_secret` are not set. |
| `ao_password`        | no       | Password for interactive login.                                             |
| `ao_access_token`     | no       | JWT bearer token. Auto-populated and cached by the `ao_auth` role. Do not set manually. |
| `ao_access_token_expiry` | no    | Epoch seconds when `ao_access_token` expires. Auto-managed by `ao_auth`. Do not set manually. |

Connection variables used by every role:

| Variable Name        | Default Value             | Description                                              |
| :-------------------- | :------------------------: | :---------------------------------------------------------- |
| `ao_hostname`          | `http://localhost:8000`    | Base URL of the AO server (no trailing slash).            |
| `ao_validate_certs`    | `true`                      | Whether to validate the AO server's SSL certificate.      |
| `ao_request_timeout`   | `10`                        | HTTP request timeout, in seconds.                          |
| `ao_state`             | `present`                   | Default state applied to objects unless overridden per-object. |

## Included Content

| Role                     | Purpose                                                       |
| :------------------------ | :--------------------------------------------------------------- |
| `global_vars`              | Shared variables (e.g. `operation_translate`) used by every role |
| `ao_auth`                  | Acquires and caches the short-lived AO bearer token              |
| `collect_async_status`     | Waits for and collects the result of async resource tasks        |
| `dispatch`                 | Orchestrates every resource role based on which variables are defined |
| `ao_settings`              | Manage runtime settings (`/api/v1/settings`)                     |
| `ao_identity_providers`    | Manage identity providers (`/api/v1/identity_providers`)         |
| `ao_projects`              | Manage projects (`/api/v1/projects`)                              |
| `ao_credentials`           | Manage credentials (`/api/v1/credentials`)                        |
| `ao_users`                 | Manage local users (`/api/v1/users`)                               |
| `ao_user_groups`           | Manage user groups (`/api/v1/groups`)                              |
| `ao_service_accounts`      | Manage service accounts (`/api/v1/service_accounts`)             |
| `ao_roles`                 | Manage custom RBAC roles (`/api/v1/roles`)                        |
| `ao_policies`              | Manage RBAC policies (`/api/v1/policies`)                         |
| `ao_role_assignments`      | Manage role assignments (`/api/v1/role_assignments`)              |
| `ao_integrations`          | Manage integrations -- MCP servers, LLM providers, etc. (`/api/v1/integrations`) |
| `ao_workflows`             | Manage workflow definitions (`/api/v1/workflows`)                 |
| `ao_tools`                 | Enable/disable discovered tools (`/api/v1/tools`, update-only)    |

The following AO API domains are intentionally **not** modeled as configuration-as-code roles because they represent runtime/operational state rather than declarative configuration: `Auth` (session lifecycle), `Admin` (maintenance operations), `AAP Proxy` (runtime passthrough), `Executions`/`Invocations` (runtime job state), `Approvals` (created internally by the workflow engine), and `Files` (binary blob uploads).

## Using This Collection

Define your connection and auth variables (e.g. in `configs/auth.yml`), then run the `configure_ao.yml` playbook:

```yaml
---
ao_hostname: https://ao.example.com
ao_client_id: "{{ vault_ao_client_id }}"
ao_client_secret: "{{ vault_ao_client_secret }}"
```

```console
ansible-playbook zjleblanc.ao_configuration.configure_ao.yml
```

Or call the `dispatch` role directly from your own playbook:

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

### Error Handling

As in `infra.aap_configuration`, resource roles run their API calls asynchronously and collect results via the `collect_async_status` role. Setting `ao_configuration_collect_logs: true` collects every failure instead of stopping on the first one; failures land in `ao_configuration_role_errors`, keyed by resource type (e.g. `ao_credentials_errors`).

### Wildcard Variables

Like `infra.aap_configuration`, you may split configuration across environment-suffixed variables (e.g. `ao_credentials_prod`, `ao_credentials_dev`) and let the `dispatch` role merge them into the base variable (`ao_credentials`) automatically by setting `dispatch_include_wildcard_vars: true`.

## License

[GPLv3+](LICENSE)

## Author

[Zack LeBlanc](https://github.com/zjleblanc)
