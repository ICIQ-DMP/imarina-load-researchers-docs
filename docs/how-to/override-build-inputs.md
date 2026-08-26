# How to override build's input files

By default, `build` reads its 9 input files from `./input/`, under fixed
names. This guide covers pointing it at files somewhere else instead —
useful for local development (e.g. files kept in sync by OneDrive for Linux
under `services/onedrive/data`), or for testing against a specific dataset
without touching `./input/`.

## Option 1: point at a different directory, same filenames

If your files are all in one directory, still under `build`'s expected
names (see [Getting started](../tutorials/getting-started.md) for the exact
list), use `--input-dir`:

```shell
./venv/bin/imarina-load-researchers build \
  --input-dir services/onedrive/data/_Projects/imarina-load-researchers/runtime/input
```

## Option 2: override individual files

Each input has its own option, which takes precedence over `--input-dir` (or
the `./input/` default) for that one file:

```shell
./venv/bin/imarina-load-researchers build \
  --a3-input path/to/some_a3_dump.xlsx \
  --imarina-input path/to/some_previous_upload.xlsx \
  --countries-dict path/to/countries.xlsx \
  --jobs-dict path/to/jobs.xlsx \
  --personal-web-path path/to/personal_web.xlsx \
  --unit-group-path path/to/unit_group.xlsx \
  --entity-type-path path/to/unit_type.xlsx \
  --job-description-entity-path path/to/job_description_entity.xlsx \
  --sex-path path/to/sex.xlsx
```

You don't have to override every file — mix and match individual overrides
with `--input-dir` for the rest:

```shell
./venv/bin/imarina-load-researchers build \
  --input-dir services/onedrive/data/_Projects/imarina-load-researchers/runtime/input \
  --a3-input /tmp/a_specific_a3_dump.xlsx
```

Here, everything except the A3 dump comes from the OneDrive-synced
directory; the A3 dump comes from wherever you point `--a3-input`.

## Where the output goes

`--output-path` controls where `build` writes the result (default:
`./output/<datetime>__icl_ag_personal_12539.xlsx`):

```shell
./venv/bin/imarina-load-researchers build --output-path /tmp/my-test-output.xlsx
```

Note that `upload`/`publish` autodetect the *latest* file in `./output/` by
default — if you redirect the output path elsewhere, you'll need to pass
`--file-path` explicitly to those commands too.

## Skipping `download` entirely during development

If you keep `services/onedrive/data` in sync with SharePoint (see the
project README's OneDrive-for-Linux setup), you can develop against
production-shaped data without ever running `download` — just point
`--input-dir` (or the individual file options) at that synced folder, run
`build`, and inspect the result locally.
