# How `build` matches researchers between A3 and iMarina

`build`'s output isn't the A3 dump translated row-by-row — it's the
*previous iMarina upload*, updated to reflect what changed in A3 since then.
This page explains that update algorithm, and the identity-matching rules it
depends on.

## Why an update, not a replacement

iMarina expects a full researcher history, not just a snapshot of who's
currently at ICIQ — a researcher who left keeps their row, closed out with
an end date. Simply re-translating the latest A3 dump every time would lose
that history. So `build` treats each run as a diff: it starts from the
previous iMarina upload, and applies whatever changed in the new A3 dump on
top of it.

## Two phases

### Phase 1 — check who's still in A3

For every researcher already in the previous iMarina upload, `build` looks
for a match in the new A3 dump (see "Matching identity" below):

- **No match found** → this person is no longer at ICIQ. Their row gets an
  end date (today's date, unless iMarina already had one) and is carried
  into the output as-is.
- **Match found, and their job/unit changed** (see "Detecting a job change"
  below) → a new row is added from the fresh A3 data, carrying the new
  position. The outgoing iMarina row for their old position is not
  separately closed out with an end date in this case — flagged as a
  possible gap between intended policy and actual behavior, worth checking
  before relying on it for someone who's changed positions more than once.
- **Match found, nothing changed** → the existing iMarina row is carried
  into the output unchanged (their contract may still be ongoing).

### Phase 2 — add anyone new

Any A3 researcher not matched to an existing iMarina row in Phase 1 is a new
hire: their row is added to the output for the first time.

## Matching identity

Two researchers (one from A3, one from iMarina) are the same person if they
share a non-empty ORCID, DNI, or email — checked in that order, first match
wins (`Researcher.is_same_person`). If more than one iMarina researcher
matches by that rule, the match is narrowed further to whichever also shares
the exact same contract start date; if that still doesn't narrow it to
exactly one, every match is kept and a warning is logged, since the
ambiguity can't be resolved automatically.

## Detecting a job change

A changed `unit_group`/`job_description` between the matched A3 and iMarina
rows normally counts as a job change — except two pairs of job titles are
treated as equivalent, not a change: "Postdoctoral researcher" /
"Associated researcher", and "Group Leader" / "Group Leader / ICREA
Professor". These exist because the same real position can be titled either
way in A3 depending on funding source, without actually being a different
job.

## Visitors are identified but not excluded

A3's export includes short-term visitors, who aren't meant to be loaded into
iMarina at all. `Researcher.is_visitor()` identifies them — primarily by A3
center code `4`, with two carve-outs (ICREA group leaders and CSC-scholarship
predoctoral researchers, both also filed under code 4 despite not being
visitors), and secondarily by contract duration for anyone else with a very
short A3 contract span. As of the current implementation, this detection is
only used to report a visitor count in the logs — visitors are **not**
actually filtered out of the generated file. If you're relying on visitors
being excluded, verify this against the current code before assuming it.

## Worked example

See [Getting started](../tutorials/getting-started.md) for a hands-on run of
this exact algorithm against a minimal dataset — one new researcher, added
in Phase 2, then correctly recognized as unchanged on a second run.
