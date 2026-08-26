# How to run the full pipeline locally

Use this when you need to reproduce (or debug) a real load run on your own
machine, instead of waiting for Jenkins. It assumes you've already completed
[Getting started](../tutorials/getting-started.md) and have SharePoint
credentials configured (see
[How to configure secrets](configure-secrets.md)).

## The command sequence

This is the same sequence Jenkins runs in production (see the repo's
`Jenkinsfile`), against a real MS List request `<OperationID>`:

```shell
rm -rf input
./venv/bin/imarina-load-researchers download <OperationID>
./venv/bin/imarina-load-researchers build --id <OperationID>
./venv/bin/imarina-load-researchers upload --id <OperationID>
./venv/bin/imarina-load-researchers notify --id <OperationID> --status success
```

A few things worth knowing before you run this:

- `download`'s `<OperationID>` is a **positional** argument, not `--id` —
  the other three commands take it as `--id`.
- `--id` on `build`/`upload` is optional everywhere else, but here it's what
  keeps the MS List item's Workflow State field in sync as each step runs.
  Passing it is what makes this a "real" run against a tracked request,
  rather than a disconnected local test.
- If any step fails, run `notify --id <OperationID> --status error` instead
  of the `success` call — that's what tells the requester (by email) that
  the run failed, and marks the request's Workflow State as `Error`.
- `rm -rf input` at the start matches what Jenkins does: `download` doesn't
  clean stale files out of `input/` itself, it only adds/overwrites the ones
  it's responsible for.

## Running just one stage

Each command works fine on its own if you already have the right files in
place — you don't have to run the whole sequence every time. For example, to
re-run just `build` against files you already downloaded:

```shell
./venv/bin/imarina-load-researchers build --id <OperationID>
```

Or to skip `download` entirely during local development, point `build`'s
inputs at files kept in sync by OneDrive for Linux instead — see
[How to override build's input files](override-build-inputs.md).

## Publishing

`publish` is deliberately **not** part of this sequence — it's a separate,
human-gated step. See
[How to publish a load file manually](publish-a-load-file-manually.md).

## Where to look when something fails

Logging is configured by a global option, so `-v`/`-l` go **before** the
subcommand, not after:

```shell
./venv/bin/imarina-load-researchers -v build --id <OperationID>
```

INFO is the default level; `-v`/`--verbose` bumps it to DEBUG, `--t`/`--trace`
to the most detailed level, and `-q`/`--quiet` or `-Q`/`--Quiet` quiet it
down. Add `-l/--log-file <path>` to also write to a file (in Jenkins/Docker
this is `logs/`, mounted as a volume — see `compose.yml`).
