# Snippets

## Vault

### Vault Subcommands
Adds the following subcommands to `vault` via `~/.bash_profile`:

```
$ vault append some/path foo=bar hello=world    # Appends secrets to a path instead of overwriting it.
$ vault cp old/path new/path                    # Copies one path to another.
$ vault delkey some/path foo                    # Deletes a specific secret from a path.
$ vault mv old/path new/path                    # Moves one path to another.
```

```
vault_subcmd() {
    if [ -z "$1" ]; then
        vault
    else
        case "$1" in
            "append")
                for ARG in "${@:3}"; do
                    vault write $2 @<(vault read -format=json $2 | jq -r --arg k "$(echo $ARG | awk -F '=' '{print $1}')" --arg v "$(echo $ARG | awk -F '=' '{print $2}')" '.data * ([{"key": $k, "value": $v}] | from_entries)')
                done
                ;;
            "cp")       vault write "$3" @<(vault read -format=json "$2" | jq -r .data) ;;
            "delkey")   vault write "$2" @<(vault read -format=json "$2" | jq ".data | del(.$3)") ;;
            "mv")       vault write "$3" @<(vault read -format=json "$2" | jq -r .data) && vault delete "$3" ;;
            *)          vault "$@"
        esac
    fi
}
alias vault="vault_subcmd"

```
