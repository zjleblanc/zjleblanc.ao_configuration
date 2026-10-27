# zjleblanc.ao_configuration.ao_auth

## Description

Acquires a short-lived AO JWT access token and caches it (plus its expiry) as host facts
for the remainder of the play. Every `ao_*` resource role depends on this role via
`meta/main.yml`; because Ansible deduplicates identical role dependencies within a single
play, the login only actually happens once per `ansible-playbook` run -- not once per
resource role.

The AO API does not issue long-lived personal access tokens, so this role re-implements
the same token lifecycle used by the AO Postman collection's pre-request script: check
whether the cached token is missing or expired (with a safety buffer), and if so,
re-authenticate.

Two authentication methods are supported. If `ao_client_id`/`ao_client_secret` are both
set, the OAuth2 `client_credentials` grant (`POST /api/v1/auth/token`) is used -- this is
the preferred method for unattended automation via a service account. Otherwise,
`ao_username`/`ao_password` are used against `POST /api/v1/auth/login`.

## Variables

| Variable Name                | Default Value             | Required | Description                                                                 |
| :----------------------------- | :--------------------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_hostname`                   | `http://localhost:8000`      | yes      | Base URL of the AO server.                                                    |
| `ao_validate_certs`             | `true`                        | no       | Whether to validate the AO server's SSL certificate.                          |
| `ao_request_timeout`            | `10`                          | no       | HTTP request timeout, in seconds.                                             |
| `ao_client_id`                  | `""`                           | no       | Service account client ID (`client_credentials` grant). Preferred for automation. |
| `ao_client_secret`              | `""`                           | no       | Service account client secret.                                                |
| `ao_username`                   | `""`                           | no       | Username for interactive login. Used only when `ao_client_id`/`ao_client_secret` are unset. |
| `ao_password`                   | `""`                           | no       | Password for interactive login.                                              |
| `ao_access_token`                | `""`                           | no       | JWT bearer token. Auto-populated and cached by this role; do not set manually except to seed a pre-existing token. |
| `ao_access_token_expiry`          | `""`                           | no       | Epoch seconds when `ao_access_token` expires. Auto-managed by this role; do not set manually. |
| `ao_auth_safety_buffer_seconds`  | `30`                          | no       | Re-authenticate this many seconds before the token's real expiry.           |

### Secure Logging Variables

| Variable Name                           | Default Value | Required | Description                                                                 |
| :----------------------------------------- | :--------------: | :------: | :----------------------------------------------------------------------------- |
| `ao_configuration_auth_secure_logging`       | `true`             | no       | Whether to suppress logging of the login/token tasks (they handle credentials). |
| `ao_configuration_secure_logging`            | `false`            | no       | Shared toggle across the whole collection; see above.                        |

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zack LeBlanc](https://github.com/zjleblanc)
