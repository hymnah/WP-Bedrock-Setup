# Bedrock Setup Script

A bash script that scaffolds a new [Roots Bedrock](https://roots.io/bedrock/) WordPress project and configures its `.env` file in one command, including unique security salts.

```bash
PROJECT_NAME="Acme Site" DB_NAME=acme_db DB_USER=acme_user ./setup-bedrock.sh
```

## Features

- Creates a fresh Bedrock project with Composer
- Derives a clean folder name from the project name (`Acme Site` → `acme-site`)
- Writes database credentials, environment, and site URL to `.env`
- Generates unique random values for all eight WordPress keys and salts
- Prompts for the database password with hidden input, so it never has to be stored in the script
- Every setting can be overridden with environment variables, no editing required
- Refuses to overwrite an existing folder
- Locks down `.env` permissions (`chmod 600`)
- Works on Linux and macOS
- Stops immediately on any error (`set -euo pipefail`)

## Requirements

- Bash
- [Composer](https://getcomposer.org/)
- OpenSSL (used to generate salts)
- PHP at the version required by the current Bedrock release
- A MySQL or MariaDB server for the site itself

The script checks for Composer and OpenSSL before doing anything and exits with a clear message if either is missing.

## Installation

Download the script and make it executable:

```bash
chmod +x setup-bedrock.sh
```

Optionally, move it somewhere on your `PATH` so you can run it from any folder:

```bash
mv setup-bedrock.sh ~/.local/bin/setup-bedrock
```

## Usage

Run it from the folder where the new project should be created.

With the defaults (you'll be prompted for the database password):

```bash
./setup-bedrock.sh
```

With your own values:

```bash
PROJECT_NAME="Acme Site" \
DB_NAME=acme_db \
DB_USER=acme_user \
WP_HOME=https://acme.test \
./setup-bedrock.sh
```

Fully non-interactive, for example in automation:

```bash
PROJECT_NAME="Acme Site" DB_PASSWORD="$ACME_DB_PASS" ./setup-bedrock.sh
```

> Avoid typing real passwords directly on the command line, since they can end up in your shell history. Let the script prompt you, or pass them from an existing environment variable.

## Configuration

| Variable       | Default                 | Description                                                  |
| -------------- | ----------------------- | ------------------------------------------------------------ |
| `PROJECT_NAME` | `Sample Project`        | Display name. Also written to `.env` as `PROJECT_NAME`.      |
| `PROJECT_DIR`  | slug of `PROJECT_NAME`  | Folder to create the project in.                             |
| `DB_NAME`      | `sample_db`             | Database name.                                               |
| `DB_USER`      | `sample_db_user`        | Database user.                                               |
| `DB_PASSWORD`  | *(prompted)*            | Database password. Asked for with hidden input if not set.   |
| `DB_HOST`      | `localhost`             | Database host.                                               |
| `WP_ENV`       | `development`           | Bedrock environment: `development`, `staging`, or `production`. |
| `WP_HOME`      | `https://sample.test`   | Full URL of the site, without a trailing slash.              |

You can also change the defaults permanently by editing the `CONFIGS` block at the top of the script.

## What it does

1. Checks that Composer and OpenSSL are installed.
2. Works out the project folder and makes sure it doesn't already exist.
3. Asks for the database password if one wasn't provided.
4. Runs `composer create-project roots/bedrock`.
5. Makes sure a `.env` file exists (Bedrock normally creates it, so this is a fallback).
6. Sets each configured value in `.env`. Commented-out keys such as `# DB_HOST=...` are uncommented, and keys that don't exist yet are appended.
7. Replaces every `generateme` placeholder with a unique random salt.
8. Restricts `.env` so only your user can read it.
9. Prints the remaining manual steps.

## After setup

The script prepares the files only. To finish:

1. Create the database and grant access to the user, for example:

   ```sql
   CREATE DATABASE acme_db;
   CREATE USER 'acme_user'@'localhost' IDENTIFIED BY 'your-password';
   GRANT ALL PRIVILEGES ON acme_db.* TO 'acme_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

2. Point your web server's document root to the project's `web/` folder, not the project root.
3. Visit `WP_HOME/wp/wp-admin` to complete the WordPress install.

## Notes and limitations

- Values can't contain a single quote (`'`), because `.env` values are written in single quotes. The script stops with an error if one does.
- The database itself is not created. See [After setup](#after-setup).
- `.env.example` is left in place on purpose. It documents the variables the project needs and should stay in version control. `.env` should never be committed.

## Troubleshooting

**`'composer' is required but not installed`**
Install Composer and make sure `composer` is on your `PATH`.

**`'acme-site' already exists`**
Delete or rename the existing folder, or choose another with `PROJECT_DIR=other-folder`.

**`Database password cannot be empty`**
Enter a password at the prompt or set `DB_PASSWORD`.

**Composer fails during create-project**
Check your PHP version and extensions against Bedrock's requirements, and run `composer diagnose`.

## Roadmap

- Create the database automatically
- Run `wp core install` with WP-CLI
- Optional starter theme with Vite, Tailwind CSS v4, and Alpine.js
- Git initialization with an initial commit
