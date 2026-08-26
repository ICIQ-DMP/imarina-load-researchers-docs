# How to configure secrets

`imarina-load-researchers` needs credentials for SharePoint/Graph API,
FTP (the iMarina server) and SMTP (notification emails). This guide covers
how to supply them, whichever environment you're running in.

## The four sources, in priority order

Every credential is read through `read_secret(name)`, which tries these
sources in order and uses the first one it finds:

1. **Docker secret** — a file at `/run/secrets/<NAME>`.
2. **Local secrets file** — `<project-root>/secrets/<NAME>`.
3. **Environment variable** — `<NAME>`.
4. **HashiCorp Vault** — see
   [Secrets and credentials](../explanation/secrets-and-credentials.md) for
   how Vault is configured and mapped.

You only need to set up *one* of these for a given secret — pick whichever
fits how you're running the app.

## The secret names

| Name | Used by |
|---|---|
| `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET`, `CLIENT_NAME` | SharePoint/Graph API auth |
| `SHAREPOINT_DOMAIN`, `SITE_NAME`, `DRIVE_ID`, `LIST_NAME` | SharePoint site/list/drive lookup |
| `FTP_HOST`, `FTP_USER`, `FTP_PASSWORD`, `FTP_PORT` | `publish`'s FTP push |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD` | `notify`'s emails |

Not every command needs every secret — `build`, for instance, needs none of
them at all, since it only transforms local files.

## Option A: local secrets files (simplest for local dev)

```shell
mkdir -p secrets
echo -n "your-tenant-id" > secrets/TENANT_ID
echo -n "your-client-id" > secrets/CLIENT_ID
echo -n "your-client-secret" > secrets/CLIENT_SECRET
# ...one file per secret you need
```

These files are read directly off disk relative to the project root — no
extra configuration needed. `secrets/` should never be committed (check it's
covered by `.gitignore`).

## Option B: environment variables

```shell
export TENANT_ID="your-tenant-id"
export CLIENT_ID="your-client-id"
export CLIENT_SECRET="your-client-secret"
./venv/bin/imarina-load-researchers download <OperationID>
```

## Option C: Docker secrets (production/CI)

`compose.yml` already wires the SharePoint and FTP secrets as Docker
secrets, each backed by a file under `./secrets/`:

```yaml
secrets:
  - CLIENT_ID
  - CLIENT_SECRET
  - TENANT_ID
  # ...

secrets:
  CLIENT_ID:
    file: ./secrets/CLIENT_ID
  # ...
```

Populate the same `secrets/<NAME>` files as in Option A, then run:

```shell
docker compose up --build
```

Docker mounts each one at `/run/secrets/<NAME>` inside the container, which
is `read_secret`'s first, highest-priority source.

## Verifying a secret is being picked up

If a secret can't be resolved from any of the four sources, the command
fails with a `SecretUnavailableError`/`MissingCredentialsError` naming which
secret was missing — check its traceback for the exact name, then confirm
that name has a file/env var/Docker secret set up for it.
