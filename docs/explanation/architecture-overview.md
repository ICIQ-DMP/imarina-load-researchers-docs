# Architecture overview

`imarina-load-researchers` is built as five small, independent CLI commands
— `download`, `build`, `upload`, `publish`, `notify` — rather than one
monolithic program. This page explains that shape, and the design decision
underneath it that makes it work (and that you need to understand before
changing any of the five).

## The five commands

| Command | Job |
|---|---|
| `download` | Populate `./input/` with every file `build` needs. |
| `build` | Transform an A3 HR dump + the previous iMarina upload into the next iMarina upload. |
| `upload` | Push the built file to SharePoint for human review. |
| `publish` | Push a reviewed file to the live iMarina server over FTP. |
| `notify` | Email the requester the outcome of a run. |

`build` is the only one that's pure data transformation — no network calls,
no credentials needed. The other four each talk to exactly one external
system: `download`/`upload` to SharePoint, `publish` to the iMarina FTP
server, `notify` to SMTP and the SharePoint-hosted request list.

## They don't call each other

This is the key design decision: none of these five commands import or
invoke any of the others. There's no in-process handoff, no shared object
passed from one stage to the next, and no manifest file describing what a
given run produced. Instead, they communicate purely through a **filesystem
naming convention**: fixed folder names (`./input/`, `./output/`) and fixed
filenames, all relative to the current working directory.

`build` doesn't know that `download` ran before it, or that `upload` will
run after it. It just reads whatever is in `./input/` under the expected
names, and writes its result to `./output/` under a name the *next* stage
knows how to find. This is what lets each command be run standalone (for
testing, for a manual fix, for local development against synced files — see
[How to override build's input files](../how-to/override-build-inputs.md))
without any of the surrounding orchestration.

## The trade-off: an implicit contract

The convention that glues the five commands together — which folder, which
filename, which datetime format — lives in one place in the code
(`core/defines.py`'s `FILENAME_PREFIX`/`FILENAME_IMARINA_SUFFIX`/
`FILENAME_A3_SUFFIX`/`DATETIME_FORMAT`/`REQUIRED_INPUT_FILES`), but nothing
*enforces* it. Renaming a default path or changing the filename format in
one command doesn't raise an error anywhere — it just silently desyncs from
what the next command expects to find. This is a known, accepted risk of the
current design: the contract is real, but implicit, and only a full run
(or the test suite) will actually surface a break in it.

## Jenkins is the orchestrator

Since the commands don't call each other, something has to run them in
order. In production, that's Jenkins: the repo's `Jenkinsfile` runs
`download` → `build` → `upload`, each wrapped in its own try/catch that
calls `notify --status error` on failure. `publish` runs from a second,
separate Jenkins pipeline (`Jenkinsfile.publish`), triggered only after a
human approves the file `upload` produced — see
[The end-to-end workflow](the-end-to-end-workflow.md) for the full,
human-and-system picture.

## "Latest" always means by filename, not by time

A recurring pattern across the codebase: whenever a command needs to find
"the latest" of a group of files (the newest A3 dump, the newest previous
iMarina upload, the newest build output to publish), it does so by parsing
the datetime encoded at the start of the filename — never by relying on
filesystem metadata. See
[Why filenames encode dates](naming-conventions.md) for why, and the one
place a modification-time fallback does exist.
