# zjleblanc.ao_configuration.global_vars

## Description

An Ansible role to define global variables that will be available to all of the
roles in the collection, if they are configured as follows:

```console
# tail -4 meta/main.yml

dependencies:
  - global_vars
```

## Provided Variables

This is currently providing the following variables:

| Variable Name         | Default Value                                                      | Required | Description                                                      |
| :--------------------- | :------------------------------------------------------------------: | :------: | :----------------------------------------------------------------- |
| `operation_translate`   | [See the default value below](#operation_translate-default-value)  | Yes      | Provides translation from object states to human interpretation |

### operation_translate Default value

```yaml
---
operation_translate:
  present:
    verb: "Create/Update"
    action: "creation"
  absent:
    verb: "Remove"
    action: "deletion"
  exists:
    verb: "Already Exists"
    action: "exists"
  enabled:
    verb: "Enabled"
    action: "enabled"
  disabled:
    verb: "Disabled"
    action: "disabled"
...
```

## License

[GPLv3+](https://github.com/zjleblanc/zjleblanc.ao_configuration/blob/main/LICENSE)

## Author

[Zach LeBlanc](https://github.com/zjleblanc)
