# n8n-clevercloud

Deploy n8n on the Clever Cloud platform.

## How to deploy

### Step 0 — Create a repository
Everything on Clever Cloud is based on Git. Create a new repository on GitHub and push your project there.

### Step 1 — Setup the code
Inside the repository add two files: `package.json` and `run.sh`.

#### package.json
```json
{
    "name": "n8n-clever-cloud",
    "version": "1.0.0",
    "description": "n8n on clevercloud",
    "dependencies": {
        "n8n": "0.125.0"
    },
    "scripts": {
        "start": "./run.sh"
    },
    "author": "Loic Tosser <wowi42>",
    "license": "ISC",
    "bugs": {
        "url": "https://github.com/wowi42/n8n-clever-cloud/issues"
    },
    "homepage": "https://github.com/wowi42/n8n-clever-cloud#readme",
    "engines": {
        "node": "^14"
    }
}
```

Key points:
- n8n is declared as a dependency.
- A start script points to `run.sh`.
- Node engine is set to `^14`.

#### run.sh
```bash
#!/bin/bash
set -e
set -x

# Clever Cloud assigns PORT, APP_ID, and PostgreSQL add-on env vars (POSTGRESQL_ADDON_*)
export N8N_PORT=$PORT
export N8N_PROTOCOL=https
export DB_TYPE=postgresdb
export DB_POSTGRESDB_DATABASE=$POSTGRESQL_ADDON_DB
export DB_POSTGRESDB_HOST=$POSTGRESQL_ADDON_HOST
export DB_POSTGRESDB_PORT=$POSTGRESQL_ADDON_PORT
export DB_POSTGRESDB_USER=$POSTGRESQL_ADDON_USER
export DB_POSTGRESDB_PASSWORD=$POSTGRESQL_ADDON_PASSWORD
export GENERIC_TIMEZONE="UTC"
export N8N_BASIC_AUTH_ACTIVE=true

# Optional: allow overriding host via N8N_HOST env var
if [ -z "$N8N_HOST" ]
then
    export N8N_HOST=$(echo "$APP_ID" | tr '_' '-').cleverapps.io
fi

echo "Host: $N8N_HOST"
env
./node_modules/.bin/n8n start
exit 1
```

Make the script executable before committing:
- chmod +x run.sh

Commit both files to your repo.

### Step 2 — Prepare Clever Cloud

#### Add a PostgreSQL add-on
1. In Clever Cloud: Create -> an add-on -> PostgreSQL -> DEV
2. Give it a name and create it.
Clever Cloud exposes PostgreSQL connection details via environment variables (POSTGRESQL_ADDON_DB, POSTGRESQL_ADDON_HOST, POSTGRESQL_ADDON_PORT, POSTGRESQL_ADDON_USER, POSTGRESQL_ADDON_PASSWORD) which `run.sh` uses.

#### Create the application
1. Create -> an application
2. Select your GitHub repository
3. Choose NodeJS runtime
4. Pick instance size (XS is usually fine for testing)
5. Give the app a name and description
6. Skip adding more add-ons here (you already created PostgreSQL)

#### Environment variables (minimum)
Configure these env vars for the application in Clever Cloud:
- N8N_BASIC_AUTH_USER — username for n8n UI
- N8N_BASIC_AUTH_PASSWORD — password for n8n UI
- N8N_ENCRYPTION_KEY — key used to encrypt credentials before storage

Optional:
- N8N_HOST — the hostname to use for n8n (defaults to the cleverapps.io domain for the app). Setting a custom host is recommended if you use your own domain.

Notes
- The run script relies on Clever Cloud's provided PostgreSQL add-on environment variables. If you use another DB provider, update the DB_* environment variables accordingly.
- The package.json pins n8n to 0.125.0; update the version as needed.
- Confirm Node version compatibility with the n8n version you choose.

License
- See package.json for license and author fields.
