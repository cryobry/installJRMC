# mc-field-copy

Recreate your custom (user) library fields on another JRiver Media Center installation without setting them up by hand. `mc-field-copy` reads field definitions from the old installation through the Media Center Web Service (MCWS) and creates only the fields missing from the new one.

The script copies field definitions, including calculated expressions. It does not copy tracks, playlists, or per-track values, and it leaves existing fields unchanged. Run it before importing your media into the new library.

## Requirements

- Python 3.8 or later. No extra packages are needed.
- **Tools > Options > Media Network > Use Media Network** enabled on each instance you connect to. Have the Media Network login handy for each server that requires one. For a live copy, both instances must be running; with a JSON snapshot, you can run them one at a time.

## Quick start

Replace the example hosts, usernames, and passwords with your own. Preview the changes with `--dry-run`. Review the proposed fields, then rerun the same command without `--dry-run` to create them. Fields are kept when the source marks them as `User` and editable.

### Two MC instances on separate machines

```sh
python3 mc-field-copy --source http://username:password@oldpc:52199 --target http://username:password@newpc:52199 --dry-run
```

---

### Two MC instances on the same computer (using different ports)

If both Media Center instances are running on the computer where you run the script, use `localhost` for both URLs and specify each instance's Media Network port. This example copies from port `52199` to port `52198`:

```sh
python3 mc-field-copy --source http://username:password@localhost:52199 --target http://username:password@localhost:52198 --dry-run
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

### One MC library at a time (save and restore fields from file)

If you can only open one library at a time, open the source library and save its field definitions to a new JSON file. The export only connects to the source and does not change it:

```sh
python3 mc-field-copy --source http://username:password@localhost:52199 --export-fields fields.json
```

Now switch Media Center to the target library. Use the JSON file as `--source` and the running instance as `--target`. Both steps can use the same server address because they run at different times:

```sh
python3 mc-field-copy --source fields.json --target http://username:password@localhost:52199 --dry-run
```

After reviewing the preview, rerun without `--dry-run`. The snapshot includes field names, types, and calculated expressions, not tracks, field values, or server credentials. Existing target fields are left unchanged.

## Options

| Option | Description |
| --- | --- |
| `-s`, `--source URL_OR_FILE` | Read field definitions from a live server or a saved JSON snapshot. |
| `--export-fields FILE` | Save field definitions from a live source to a new JSON file, without connecting to a target. |
| `-u`, `--target URL` | Create missing fields on this server. |
| `-n`, `--dry-run` | Preview the fields that would be created. |
| `-f`, `--field FIELD=TYPE` | Add a missing field or override its type: `string`, `integer`, `list`, or `date`. Repeat for multiple fields. |
| `-m`, `--mpl PATH ...` | Use [MPL files or directories](#mpl-fallback) to select fields. |

## `--source` and `--target` URLs with username and password

Use the same URL format for `--source` and `--target`. Each server can have its own credentials.

| URL | Login |
| --- | --- |
| `http://username:password@newpc:52199` | user `username` with password `password` |

If your password contains special characters, percent-encode them: for example, `p@ss` becomes `p%40ss`.

## Field types

The script preserves types that MCWS can create and maps the others to supported types:

| Source type | Created as |
| --- | --- |
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
python3 mc-field-copy --target http://username:password@localhost:52199 --mpl ~/exports/ --dry-run
```

MPL files provide field names and values, not types or formulas. The first matching rule wins:

| What the script finds | Type used |
| --- | --- |
| A matching definition from `--source` | [Source type](#field-types), with its formula if present |
| A single-line value containing `;` | `list` |
| At least one nonempty value, and all nonempty values are whole numbers | `integer` |
| Anything else, including fields with only empty values | `string` |

Use `--field` to override a type or add a field missing from the export.

You can combine `--mpl` with `--source fields.json`. The MPL selects which fields to copy, and the saved JSON supplies types and calculated expressions for matching names. Fields without a match use the inference rules above.

```sh
python3 mc-field-copy --source fields.json --mpl playlist.mpl --target http://username:password@localhost:52199 --dry-run
```
