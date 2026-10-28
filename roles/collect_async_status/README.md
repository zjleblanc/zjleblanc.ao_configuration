# zjleblanc.ao_configuration.collect_async_status

## Description

An Ansible role that waits for an async task (started with `async`/`poll: 0` by a resource
role) to finish, optionally registers its result, and -- when it failed -- either stops the
play immediately or appends the failure to `ao_configuration_role_errors` so the play can
continue and report every failure at the end (see the `dispatch` role).

This role is an internal dependency of every `ao_*` resource role in this collection; it is
not intended to be used directly.

## Variables

| Variable Name           | Required | Description                                                                 |
| :------------------------ | :------: | :----------------------------------------------------------------------------- |
| `cas_job_async_results_item` | yes   | One element of the `results` list returned by the async-launched task.       |
| `cas_secure_logging`        | yes    | Whether to suppress logging of the async status check (`no_log`).           |
| `cas_async_retries`         | yes    | Number of times to poll `async_status` before giving up.                     |
| `cas_async_delay`           | yes    | Delay, in seconds, between polls.                                            |
| `cas_register_subvar`       | yes    | Key under `ao_configuration_register` to append this result to.             |
| `cas_error_list_var_name`   | yes    | Key under `ao_configuration_role_errors` to append failures to.             |
| `cas_object_label`          | yes    | Human-readable label used for the `async_status` task name.                 |

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
