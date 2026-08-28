# How to request an iMarina researcher load

This guide is for anyone at ICIQ who needs the researcher list in iMarina
refreshed — you don't need to know anything about the code or the CLI to
follow it.

At a glance, the process is:

1. Put the two input files on SharePoint, correctly named (this guide,
   steps 2 and 3).
2. Submit the request form, linking those files (step 4).
3. Wait for the automated run, then review, correct and approve (or
   reject) the result (steps 5–7).

Steps 2 and 3 happen **before** you open the form. Everything from step 5
on happens by email and Microsoft Approval, with no further action needed
until you're asked to review.

## 1. Who can do this

Triggering this workflow is restricted to a small, named set of people, for
data-protection reasons (see
[Data protection and GDPR](../explanation/data-protection.md)):

- Dr. Sonia Sayalero, responsible for Institutional Strengthening operations
  at ICIQ, and the Severo Ochoa administrator.
- Aleix Mariné-Tena, ICIQ's data steward, who designed and maintains this
  workflow.
- An apprentice, if this project is assigned to one.

If you're not one of these people and you need a load run, ask one of them.

Note also that the raw A3 HR export can only come from **Human Resources**
— specifically Mara Cruz, the head of HR, who is the only authorized source
of the A3 extract. You never generate the A3 file yourself; you ask HR for
it. HR provides one at most once a month.

## 2. Put the input files on SharePoint

The load is built from two input files:

- **The A3 HR database export** — a bulk snapshot of ICIQ personnel data,
  in A3 format, obtained from HR (see step 1).
- **The previous iMarina upload** — the last spreadsheet that was published
  to iMarina, used as the baseline the new one is built on top of.

Both live on the **Institutional Strengthening** SharePoint site, under
`_Projects/imarina-load-researchers/runtime/`. Before you fill in the
request form, make sure the version of each file you want the load built
from is sitting in the right folder, under the right name. The two files go
to two different folders and follow two slightly different naming rules —
they're covered separately below.

Both files are technically **optional** on the form: if you don't link one,
the system falls back to the latest matching file it already has (see
"When you can skip a file" at the end of this step). But the fallback can
only ever pick a file that is *already* on SharePoint under the correct
name — so if you have a newer file, it has to be uploaded here first
regardless.

**Names are checked automatically.** Whenever a file lands in `runtime/a3`
or `runtime/imarina`, an automated check validates its name against the
rules below. If the name doesn't match, whoever last modified the file is
notified and the file is effectively ignored by the pipeline. Get the name
exactly right, including the double underscore `__` before the descriptive
part.

### 2a. The A3 HR export

- **Folder:** `_Projects/imarina-load-researchers/runtime/a3`
- **Name:** `{DATETIME}__listado_personal_A3.xlsx`

`{DATETIME}` is when HR actually took the snapshot, formatted
`YYYY-MM-DD_HH-mm-ss` — four-digit year, then month, day, hour, minute,
second, with `-` between the parts of the date and of the time and a single
`_` between them. Example:

```
2025-03-12_12-00-00__listado_personal_A3.xlsx
```

If you don't know the exact time the export was taken — which is normal,
since it's produced manually — use `12-00-00` for the time part. Precision
below the day doesn't matter here: there's at most one A3 dump a month, so
the date alone is enough to tell them apart.

Upload the file HR gave you into that folder, renamed to match. This is
also the folder the fallback reads from, so there's nothing else to copy or
move — once it's here under the right name, it's in position.

### 2b. The previous iMarina upload

- **Folder:** `_Projects/imarina-load-researchers/runtime/imarina`
- **Name:** `{DATETIME}__icl_ag_personal_12539.xlsx`

`{DATETIME}` follows the exact same `YYYY-MM-DD_HH-mm-ss` convention as the
A3 file, and here it represents when that iMarina spreadsheet was produced.
Example:

```
2026-05-01_12-00-00__icl_ag_personal_12539.xlsx
```

You normally do **not** need to touch this file — the baseline should
almost always just be the last thing that was published, which the fallback
picks up on its own (from a different folder, `runtime/published` — see
below). Only upload a file into `runtime/imarina` when you deliberately
want the new load built on top of a *specific* earlier iMarina file rather
than the most recently published one.

### When you can skip a file on the form

If you leave a file's link blank on the request form, the system falls back
to the most recent matching file it already has on SharePoint:

| File you skip | Fallback picks the latest from |
|---|---|
| A3 HR export | `runtime/a3` |
| Previous iMarina upload | `runtime/published` (the archive of every file ever published to iMarina) |

"Most recent" is decided from the `{DATETIME}` in each filename, not from
upload dates — which is exactly why the naming rules above matter.

In practice:

- **A3:** supply a link whenever HR has just handed you a fresh export.
  After you've uploaded it to `runtime/a3` (step 2a), you can either link
  it explicitly on the form or just leave the field blank — since it's now
  the newest file in that folder, the fallback will select it either way.
  Linking it explicitly is clearer and leaves less room for surprise.
- **Previous iMarina upload:** leave this blank almost every time. The
  fallback's "latest published file" is the correct baseline for a normal
  run. Only link a file here if you did the optional upload in step 2b for
  a specific reason.

## 3. Get a sharing link for each file you'll link

For every file you intend to reference on the form (see step 2), open it in
SharePoint, create a **sharing link**, and keep it handy — you'll paste it
into the form in the next step. A link restricted to people inside ICIQ is
fine; the pipeline authenticates as itself and doesn't rely on the link
being public.

You only need links for files you're linking explicitly. If you're relying
on the fallback for a file, you don't need its link.

## 4. Fill in the request form

Open the **iMarina load request** Microsoft Form and submit it. It has two
optional link fields:

- **A3 Excel input link** — paste the sharing link to the A3 export you
  placed in `runtime/a3` (step 2a / step 3). Leave blank to use the latest
  A3 file already there.
- **iMarina Excel input link** — paste the sharing link to the previous
  iMarina upload you placed in `runtime/imarina` (step 2b / step 3). Leave
  blank to use the latest published file (the normal case).

Submitting the form creates a tracked request with its own ID and starts
the automated pipeline. Everything after this point runs on its own.

## 5. Wait for the automated run

Nothing for you to do here. The pipeline downloads the inputs, builds the
new spreadsheet and uploads it to a SharePoint review folder. See
[The end-to-end workflow](../explanation/the-end-to-end-workflow.md) for
what happens behind the scenes.

When it finishes you'll get **an email with a link to the generated file**,
and **a Microsoft Approval request** asking whether to publish it to the
live iMarina server.

## 6. Review the generated file

Open the generated file from the email and check it looks right — new hires
present, people who've left flagged with an end date, changed positions
reflected.

You then have three options:

- **No errors found** — approve the request as-is, so the generated file
  gets published to iMarina unchanged.
- **Fixable errors found** — correct them directly in that same SharePoint
  file, save it, then approve the request. Whatever is in the file at the
  moment you approve is what gets published, so your corrections are
  carried through automatically; there's no separate step to resubmit a
  fixed file.
- **Too many errors, or you're not satisfied with the result** — reject the
  approval instead (see step 7).

## 7. Approve or reject

- **Approve** the Microsoft Approval to publish the file to iMarina —
  either as generated, or with any corrections you saved into it in step 6.
  You'll get a second email once the publish has actually gone through.
- **Reject** it if something looks wrong and isn't worth fixing by hand. No
  file is published, and you can investigate — or ask HR for a corrected A3
  export, redo steps 2–4, and submit a new request — without anything
  having reached the live iMarina server.

## If something goes wrong

If any step of the automated run fails, you'll get a failure email instead
of a review request. Forward it to the Digitalization team
(`digitalitzacio@iciq.es`) along with the request ID mentioned in the
email.

Common causes worth ruling out first:

- **A misnamed input file.** If you uploaded a file in step 2 but got a
  separate notification that its name is invalid, the pipeline couldn't
  see it. Fix the name (step 2a / 2b) and submit a new request.
- **Linking a file that isn't in `runtime/a3` or `runtime/imarina`.** The
  form links should point at files in those folders. A link to a copy
  sitting somewhere else on SharePoint may not be reachable by the
  pipeline.
