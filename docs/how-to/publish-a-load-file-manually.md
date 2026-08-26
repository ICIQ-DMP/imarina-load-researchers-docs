# How to publish a load file manually

`publish` is the one command that actually pushes a file to the live iMarina
FTP server. It's intentionally never run by the automated CI pipeline — it's
a manual, human-gated step, normally triggered by someone approving the
Microsoft Approval that `upload` sets in motion (see
[The end-to-end workflow](../explanation/the-end-to-end-workflow.md)). This
guide is for running it directly, by hand.

## Safety: dry-run is the default

`--dry-run` defaults to `true`. Running `publish` with no arguments connects
to the FTP server and validates the connection, but **does not** upload
anything:

```shell
./venv/bin/imarina-load-researchers publish
```

You must explicitly opt in to a real push:

```shell
./venv/bin/imarina-load-researchers publish --dry-run false
```

## Choosing which file gets published

Three ways to tell `publish` which file to send, in the order it tries them:

1. **`--file-path <path>`** — publish exactly this file, no autodetection.
2. **`--id <OperationID>`**, no `--file-path` — download the file from that
   request's "iMarina Excel output link" field on the MS List (i.e. what
   `upload` most recently pushed for that request).
3. **Neither** — autodetect the newest file in `./output/`, first by parsing
   the datetime encoded in its filename, falling back to file-modification
   time only if that fails.

```shell
# Publish a specific request's reviewed file
./venv/bin/imarina-load-researchers publish --id 66 --dry-run false

# Publish an explicit file, bypassing the MS List entirely
./venv/bin/imarina-load-researchers publish --file-path output/2026-05-01_12-00-00__icl_ag_personal_12539.xlsx --dry-run false
```

## A note on `--file-path` with no `--id`

This works, but it publishes with **no record kept anywhere** — no MS List
item gets its Workflow State or published-link field updated, so nobody
looking at the request tracker will know this publish happened. A warning is
logged when you do this; only use it if you're deliberately publishing
outside the tracked-request workflow (e.g. a one-off fix), and be ready to
explain what you did if someone asks later.

## What happens on success

A successful (non-dry-run) publish always archives the file to
`runtime/published` on SharePoint, regardless of whether `--id` was given —
that archive is what the next `download`'s fallback logic reads "the latest
published file" from. If `--id` was given, the MS List item's "iMarina Excel
published link" field and Workflow State (`Published`) are also updated.

Re-running `publish` for the same request is safe — a duplicate/accidental
second publish just re-archives and re-sets the same "Published" state, with
no other side effect.
