# Secrets and credentials

Every credential this app needs — SharePoint/Graph API, FTP, SMTP — is read
through one function, `read_secret(name)`. This page explains the reasoning
behind that design; see
[How to configure secrets](../how-to/configure-secrets.md) for the
step-by-step version.

## One function, four sources, in priority order

`read_secret` tries, in order:

1. A Docker secret at `/run/secrets/<name>`.
2. A local file at `<project-root>/secrets/<name>`.
3. An environment variable.
4. HashiCorp Vault.

The same function serves every environment the app runs in — a developer's
laptop, a CI job, a production Docker container — without any of them
needing special-casing in application code. Each source's "not found" is a
distinct, expected exception type; `read_secret` narrows its handling to
exactly those, so a genuine bug inside one source (a malformed Vault
response, say) surfaces as an error instead of being silently treated as
"just try the next source."

## Vault is the last resort, not the primary source

Not every secret needs a Vault mapping — `core/vault.py`'s `_SECRET_MAP`
only lists the ones that are actually Vault-backed in production. A secret
with no entry there simply means "this one is never resolved from Vault,"
which is a normal, common case (most local/dev secrets go through the
file or environment-variable sources instead), not a bug to fix.

When Vault *is* used, its own credentials (the token, or an AppRole
role/secret ID pair, plus the Vault address) are themselves read through the
same four-source cascade — so bootstrapping Vault access doesn't require a
separate, special credential-loading mechanism.

## Why Docker secrets outrank everything else

Docker secrets are the most production-appropriate source — mounted
read-only, never visible in `docker inspect` or process listings the way an
environment variable would be — so they're checked first. This means a
correctly configured production container "just works" without the
application needing to know it's running in Docker specifically; the
priority order does that job.

## Redaction

Every secret this app can currently resolve (from any of the four sources)
is collected once, at CLI startup, and used to configure a logging filter
that replaces any occurrence of a secret value in a log message with
`*****`. A secret that isn't configured on the current machine is simply
skipped for this purpose — it can't leak into logs if it was never read.
