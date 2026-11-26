# mc-field-copy

Recreate your custom (user) library fields on another JRiver Media Center installation without setting them up by hand. `mc-field-copy` reads field definitions from the old installation through the Media Center Web Service (MCWS) and creates only the fields missing from the new one.

The script copies field definitions, including calculated expressions. It does not copy tracks, playlists, or per-track values, and it leaves existing fields unchanged. Run it before importing your media into the new library.

## Requirements

- Python 3.8 or later. No extra packages are needed.
- Both Media Center instances running with **Tools > Options > Media Network > Use Media Network** enabled. Have the Media Network login handy for each server that requires one.

## Quick start

Replace the example hosts, usernames, and passwords with your own. Preview the changes with `--dry-run`. Review the proposed fields, then rerun the same command without `--dry-run` to create them. Fields are kept when the source marks them as `User` and editable.

### MC instances on separate machines

```sh
python3 mc-field-copy --source http://username:password@oldpc:52199 --target http://username:password@newpc:52199 --dry-run
python3 mc-field-copy --source http://username:password@oldpc:52199 --target http://username:password@newpc:52199
```

### MC instances on the same computer (using different ports)

If both Media Center instances are running on the computer where you run the script, use `localhost` for both URLs and specify each instance's Media Network port. This example copies from port `52199` to port `52198`:

```sh
python3 mc-field-copy --source http://username:password@localhost:52199 --target http://username:password@localhost:52198 --dry-run
python3 mc-field-copy --source http://username:password@localhost:52199 --target http://username:password@localhost:52198
```

#### Launching a second instance as a separate user on Linux

Use containers or TigerVNC `vncserver`, create a separate user account and start Media Center in its own VNC session. Use an unused display: this example uses `:1`. If prompted, set a VNC password for the new account.

```sh
sudo useradd --create-home jriver-test
sudo -iu jriver-test
vncserver :3 -localhost yes -geometry 1440x900 -autokill \
	-xstartup /usr/bin/mediacenter36
exit
```

Back on your normal desktop, connect to the new session using the same display number:

```sh
vncviewer localhost:3
```

In the second instance, open **Tools > Options > Media Network**, enable **Use Media Network**, and set its port to a different unused port, such as `52198`. The VNC display number is only for connecting to the desktop; use the Media Network ports in `--source` and `--target`.

## Options

| Option | Description |
|---|---|
| `-s`, `--source URL` | Read field definitions from this server. Required for direct copying. |
| `-u`, `--target URL` | Create missing fields on this server. |
| `-n`, `--dry-run` | Preview the fields that would be created. |
| `-f`, `--field FIELD=TYPE` | Add a missing field or override its type: `string`, `integer`, `list`, or `date`. Repeat for multiple fields. |
| `-m`, `--mpl PATH ...` | Use [MPL files or directories](#mpl-fallback) to select fields. |

## `--source` and `--target` URLs with username and password

Use the same URL format for `--source` and `--target`. Each server can have its own credentials.

| URL | Login |
|---|---|
| `http://myuser:secret@newpc:52199` | `myuser` with password `secret` |

If your password contains special characters, percent-encode them: for example, `p@ss` becomes `p%40ss`.

## Field types

The script preserves types that MCWS can create and maps the others to supported types:

| Source type | Created as |
|---|---|
| String, Integer, List | Same type |
| Date (float), like the `Date` field | `date` |
| Date (Unix timestamp), like `Last Played` | `integer` |
| Decimal, Path and others | `string` |

Use `--field "Moods=list"` with `--source` or `--mpl` to create `Moods` as a list, even if it wasn't in the source or export. If the source provides a calculated expression for `Moods`, the script keeps it. If `Moods` already exists on the target, the script leaves it unchanged.

You can also create fields without a source or MPL file. For example, preview adding two fields to the local target:

```sh
python3 mc-field-copy -f "Moods=list" --field "Review Date=date" --dry-run
```

## Calculated fields

A calculated field derives its value from a formula, such as `[Artist] - [Name]`. The script reads the `Expression` attribute from the source's `Library/Fields` response:

```xml
<Field Name="Artist - Title" Expression="[Artist, 1] - [Name, 1]" DataType="String" />
```

The script sends that expression to `Library/CreateField` on the target.

## MPL fallback

If the old server is unavailable, use `--mpl` to find field names in exported playlists. Pass one or more files or directories. Directories are searched recursively for `.mpl` files.

```sh
python3 mc-field-copy --target http://myuser@newpc:52199 --mpl ~/exports/ --dry-run
```

MPL files provide field names and values, not types or formulas. The first matching rule wins:

| What the script finds | Type used |
|---|---|
| A matching definition from `--source` | [Source type](#field-types), with its formula if present |
| A single-line value containing `;` | `list` |
| At least one nonempty value, and all nonempty values are whole numbers | `integer` |
| Anything else, including fields with only empty values | `string` |

Use `--field` to override a type or add a field missing from the export.
