---
comments: true
---

The following document lists the commands that the CLI currently supports.

!!! note
    Use `--help` as a flag on any command for more information.

## Usage

Call a command on the `miden-client` like this:

```sh
miden <command> <flags> <arguments>
```

Optionally, you can include the `--debug` flag to run the command with debug mode, which enables debug output logs from scripts that were compiled in this mode:

```sh
miden --debug <flags> <arguments>
```

Note that the debug flag overrides the `MIDEN_DEBUG` environment variable.

## Commands

### `init` 

Creates a configuration file for the client in the current directory.
 
```sh
# This will create a config using default values
miden init

# You can use the --rpc flag to override the default rpc config
miden init --rpc 18.203.155.106
# You can specify the port
miden init --rpc 18.203.155.106:8080
# You can also specify the protocol (http/https)
miden init --rpc https://18.203.155.106
# You can specify both
miden init --rpc https://18.203.155.106:1234

# You can use the --store_path flag to override the default store config
miden init --store_path db/store.sqlite3

# You can provide both flags
miden init --rpc 18.203.155.106 --store_path db/store.sqlite3
```

### `account` 

Inspect account details.

#### Action Flags

| Flags           | Description                                         | Short Flag|
|-----------------|-----------------------------------------------------|-----------|
|`--list`         | List all accounts monitored by this client          | `-l`      |
|`--show <ID>`    | Show details of the account for the specified ID    | `-s`      |
|`--default <ID>` | Manage the setting for the default account          | `-d`      |

The `--show` flag also accepts a partial ID instead of the full ID. For example, instead of:

```sh
miden account --show 0x8fd4b86a6387f8d8
```

You can call:

```sh
miden account --show 0x8fd4b86
```

For the `--default` flag, if `<ID>` is "none" then the previous default account is cleared. If no `<ID>` is specified then the default account is shown.

### `new-wallet`

Creates a new wallet account.

This command has two optional flags:
- `--storage-type <TYPE>`: Used to select the storage type of the account (off-chain if not specified). It may receive "off-chain" or "on-chain".
- `--mutable`: Makes the account code mutable (it's immutable by default).

After creating an account with the `new-wallet` command, it is automatically stored and tracked by the client. This means the client can execute transactions that modify the state of accounts and track related changes by synchronizing with the Miden node.

### `new-faucet`

Creates a new faucet account.

This command has two optional flags:
- `--storage-type <type>`: Used to select the storage type of the account (off-chain if not specified). It may receive "off-chain" or "on-chain".
- `--non-fungible`: Makes the faucet asset non-fungible (it's fungible by default).

After creating an account with the `new-faucet` command, it is automatically stored and tracked by the client. This means the client can execute transactions that modify the state of accounts and track related changes by synchronizing with the Miden node.

### `info`

View a summary of the current client state.

### `notes`

View and manage notes.

#### Action Flags

| Flags             | Description                                                 | Short Flag |
|-------------------|-------------------------------------------------------------|------------|
|`--list [<filter>]`| List input notes                                            | `-l`       |
| `--show <ID>`     | Show details of the input note for the specified note ID    | `-s`       |

The `--list` flag receives an optional filter:
    - pending: Only lists pending notes.
    - commited: Only lists commited notes.
    - consumed: Only lists consumed notes.
    - consumable: Only lists consumable notes. An additional `--account-id <ID>` flag may be added to only show notes consumable by the specified account.
If no filter is specified then all notes are listed.

The `--show` flag also accepts a partial ID instead of the full ID. For example, instead of:

```sh
miden notes --show 0x70b7ecba1db44c3aa75e87a3394de95463cc094d7794b706e02a9228342faeb0 
```

You can call:

```sh
miden notes --show 0x70b7ec
```

### `sync`

Sync the client with the latest state of the Miden network. Shows a brief summary at the end.

### `tags`

View and add tags.

#### Action Flags

| Flag            | Description                                                 | Aliases |
|-----------------|-------------------------------------------------------------|---------|
| `--list`        | List all tags monitored by this client                      | `-l`    |
| `--add <tag>`   | Add a new tag to the list of tags monitored by this client  | `-a`    |
| `--remove <tag>`| Remove a tag from the list of tags monitored by this client | `-r`    |

### `tx`

View transactions.

#### Action Flags

| Command | Description                                              | Aliases |
|---------|----------------------------------------------------------|---------|
| `--list`| List tracked transactions                                | -l      |

After a transaction gets executed, two entities start being tracked:

- The transaction itself: It follows a lifecycle from `pending` (initial state) and `committed` (after the node receives it).
- Output notes that might have been created as part of the transaction (for example, when executing a pay-to-id transaction).

### Transaction creation commands
#### `mint`

Creates a note that contains a specific amount tokens minted by a faucet, that the target Account ID can consume.

Usage: `miden mint --target <TARGET ACCOUNT ID> --faucet <FAUCET ID> <AMOUNT> --note-type <NOTE_TYPE>`

#### `consume-notes`

Account ID consumes a list of notes, specified by their Note ID.

Usage: `miden consume-notes --account <ACCOUNT ID> [NOTES]`

For this command, you can also provide a partial ID instead of the full ID for each note. So instead of

```sh
miden consume-notes --account <some-account-id> 0x70b7ecba1db44c3aa75e87a3394de95463cc094d7794b706e02a9228342faeb0 0x80b7ecba1db44c3aa75e87a3394de95463cc094d7794b706e02a9228342faeb0
```

You can do:

```sh
miden consume-notes --account <some-account-id> 0x70b7ecb 0x80b7ecb
```

#### `send`

Sends assets to another account. Sender Account creates a note that a target Account ID can consume. The asset is identifed by the tuple `(FAUCET ID, AMOUNT)`. The note can be configured to be recallable making the sender able to consume it after a height is reached.

Usage: `miden send --sender <SENDER ACCOUNT ID> --target <TARGET ACCOUNT ID> --faucet <FAUCET ID>  --note-type <NOTE_TYPE> <AMOUNT> <RECALL_HEIGHT>`

#### `swap`

The source account creates a Swap note that offers some asset in exchange for some other asset. When another account consumes that note, it'll receive the offered amount and it'll have the requested amount removed from its assets (and put into a new note which the first account can then consume). Consuming the note will fail if the account doesn't have enough of the requested asset. 

Usage:  `miden swap --source <SOURCE ACCOUNT ID> --offered_faucet <OFFERED FAUCET ID> --offered_amount <OFFERED AMOUNT> --requested_faucet <REQUESTED FAUCET ID> --requested_amount <REQUESTED AMOUNT> --note-type <NOTE_TYPE>`

#### Tips
For `send` and `consume-notes`, you can omit the `--sender` and `--account` flags to use the default account defined in the [config](./cli-config.md). If you omit the flag but have no default account defined in the config, you'll get an error instead.

For every command which needs an account ID (either wallet or faucet), you can also provide a partial ID instead of the full ID for each account. So instead of

```sh
miden send --sender 0x80519a1c5e3680fc --target 0x8fd4b86a6387f8d8 --faucet 0xa99c5c8764d4e011 100
```

You can do:

```sh
miden send --sender 0x80519 --target 0x8fd4b --faucet 0xa99c5 100
```

#### Transaction confirmation

When creating a new transaction, a summary of the transaction updates will be shown and confirmation for those updates will be prompted:

```sh
miden tx new ...

TX Summary:

...

Continue with proving and submission? Changes will be irreversible once the proof is finalized on the rollup (Y/N)
```

This confirmation can be skipped in non-interactive environments by providing the `--force` flag (`miden send --force ...`):

### `import`

Import entities managed by the client, such as accounts and notes. The type of entitie is inferred.

When importing notes the CLI verifies that they exist on chain. The user can add an optional flag `--no-verify` that skips this verification.

### `export`

Export input note data to a binary file .
