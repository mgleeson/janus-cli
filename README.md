# janus-cli

`janus-cli` is a small Bash CLI tool for recursive literal find/replace operations across:

- text file contents
- filenames
- directory names

It is intended for controlled site or project renames, such as changing an old hostname to a new hostname across Apache vhost files, webroots, and application config files.

## Safety warning

This script can modify many files and paths recursively. Always run a dry run first and keep backups before running it against important locations such as `/etc`, `/var/www`, or production application directories.

## Requirements

- Bash
- `find`
- `grep`
- `perl`
- standard core utilities such as `mv`, `dirname`, and `basename`

## Usage

```bash
janus-cli OLD NEW [ROOT]
```

Arguments:

- `OLD`: the literal string to search for. This must not be empty.
- `NEW`: the replacement string. This may be empty if you explicitly want to remove `OLD`.
- `ROOT`: the root directory to process. Defaults to the current directory.

If running directly from the repository checkout:

```bash
./janus oldtext newtext .
```

## Environment options

| Option | Default | Description |
| --- | --- | --- |
| `DRYRUN=1` | off | Preview changes without modifying files or renaming paths. |
| `RENAME_ROOT=1` | off | Allow the root directory itself to be renamed if its basename contains `OLD`. |
| `EXCLUDE_GIT=0` | `.git` excluded | Include `.git` directories. By default, `.git` directories are skipped. |

## Examples

Preview changes under the current directory:

```bash
DRYRUN=1 ./janus oldtext newtext .
```

Apply changes under the current directory:

```bash
./janus oldtext newtext .
```

Preview a hostname change in Apache vhost files:

```bash
DRYRUN=1 ./janus old.example.com new.example.com /etc/apache2/sites-available
```

Apply a hostname change in Apache vhost files:

```bash
sudo ./janus old.example.com new.example.com /etc/apache2/sites-available
```

Preview a change under a webroot:

```bash
DRYRUN=1 ./janus old.example.com new.example.com /var/www
```

Allow the root directory itself to be renamed:

```bash
sudo RENAME_ROOT=1 ./janus old.example.com new.example.com /var/www/old.example.com
```

Include `.git` directories deliberately:

```bash
EXCLUDE_GIT=0 ./janus oldtext newtext .
```

## Root rename behaviour

By default, `janus-cli` does not rename the root directory supplied as `ROOT`.

For example, this command processes content and child paths under `/var/www/old.example.com`, but does not rename `/var/www/old.example.com` itself:

```bash
./janus old.example.com new.example.com /var/www/old.example.com
```

To allow the root directory itself to be renamed, use:

```bash
RENAME_ROOT=1 ./janus old.example.com new.example.com /var/www/old.example.com
```

This default is intentional so the top-level target path does not unexpectedly disappear during a normal run.

## `.git` handling

By default, `.git` directories are skipped. This helps avoid corrupting repository internals such as objects, refs, hooks, and metadata.

To include `.git` directories, set:

```bash
EXCLUDE_GIT=0
```

## Line endings

The script should use Unix LF line endings. If it was edited on Windows and fails with an error similar to:

```text
/usr/bin/env: 'bash\r': No such file or directory
```

convert it back to Unix line endings:

```bash
sed -i 's/\r$//' janus
```

or, if available:

```bash
dos2unix janus
```

## Apache and Moodle notes

For an Apache and Moodle hostname change, `janus-cli` can help update filesystem-level references such as:

- Apache vhost config content
- Apache vhost filenames
- Moodle `config.php`
- webroot directory names
- hard-coded references in local project files

It does not update Moodle database content. After changing `$CFG->wwwroot`, use Moodle's supported search-and-replace tooling for database-stored URLs, then purge Moodle caches.

If an Apache vhost config file is renamed, the matching symlink in `sites-enabled` will not automatically follow the renamed file. Re-enable the site using the normal Apache tools, for example:

```bash
sudo a2dissite old.example.com.conf
sudo a2ensite new.example.com.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

## Recommended workflow

1. Back up the files and database involved.
2. Run with `DRYRUN=1`.
3. Review the planned content changes and renames.
4. Run without `DRYRUN=1`.
5. Run any application-specific follow-up tasks, such as Apache reloads or Moodle database URL replacement.
6. Search for remaining references to the old string.

## Limitations

`janus-cli` performs literal string replacement in text files and path basenames. It does not understand application semantics, database contents, symlinks that point to renamed files, external service configuration, DNS, SSL certificates, cron jobs, or reverse proxy rules.
