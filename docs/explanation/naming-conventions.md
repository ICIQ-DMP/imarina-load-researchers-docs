# Why filenames encode dates

Across this codebase, whenever something needs to find "the latest" of a
group of files — the newest A3 dump, the newest previous iMarina upload, the
newest build output to publish — it does so by parsing a datetime encoded at
the start of the filename, not by looking at filesystem timestamps. This
page explains why, and where the one exception is.

## The convention

Every file this pipeline produces or consumes is named
`{DATETIME}__<rest of the name>`, where `DATETIME` is `YYYY-MM-DD_HH-mm-ss`
(e.g. `2025-03-12_12-00-00`). Two suffixes distinguish the two kinds of
dated file the pipeline deals with:

- `__listado_personal_A3.xlsx` — an A3 HR dump.
- `__icl_ag_personal_12539.xlsx` — an iMarina upload (either the previous
  one supplied as input, or the one `build` just produced as output — same
  suffix, since from iMarina's perspective they're the same kind of file at
  different points in time).

`DATETIME` represents *when the underlying data was produced* — when HR
took the A3 snapshot, or when `build` generated the spreadsheet — not when
the file happened to land on a particular filesystem.

## Why not just use file modification time?

Modification time is a property of wherever the file currently sits, not of
the data itself. A file downloaded, re-uploaded, or synced between
SharePoint and a local machine (as `download`'s fallbacks and the
OneDrive-for-Linux dev setup both do) picks up a *new* modification time
every time it moves, even though nothing about the data changed. Relying on
mtime to pick "the latest" file would make that answer depend on which copy
of a file you happened to be looking at, and in what order files were last
touched — not on which one is actually most recent.

Encoding the real datetime in the filename instead means "latest" is a
property of the data, portable across every copy of the file, in every
location it passes through.

## The one place mtime is still used

Locally, `upload` autodetects a `./output/*.xlsx` file to push by
**modification time**, not by parsing the filename — the one deliberate
exception to this rule. `publish`, on the other hand, first tries to parse
the datetime out of the filename when picking a local file, and only falls
back to mtime if none of the candidate filenames parse.

This split exists because `./output/` is a purely local, single-machine
directory that only `build` ever writes into directly — there's no
multi-copy, multi-location scenario for `upload` to protect against there,
so the extra parsing step buys nothing. `publish`'s filename-first
preference is the more defensive choice of the two, and is arguably the more
correct pattern to follow if this convention is ever revisited.

## Remote "latest" has no mtime fallback at all

`select_latest_remote_file` — the function backing `download`'s A3/iMarina
fallbacks, which pick the newest matching file straight out of a SharePoint
folder — has **no** modification-time fallback whatsoever. A SharePoint
item's `lastModifiedDateTime` doesn't carry the same "this is when the data
was produced" meaning that local mtime does for `upload`'s narrow case
above; using it here would reintroduce exactly the "latest according to
which copy" problem this whole convention exists to avoid. If no filename in
a remote folder parses as a dated file, that's treated as an error rather
than silently falling back to something less meaningful.
