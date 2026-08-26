# Data protection and GDPR

This workflow moves personal data of ICIQ personnel — the A3 HR database
dump, and the iMarina upload derived from it — between systems with **no
human review of individual records** along the way (a human only reviews the
generated *file* before approving publication, not each row in it). That
combination means the design has to actively satisfy GDPR's purpose
limitation, data minimization and storage limitation principles, not just
describe who's allowed to click the button.

## Purpose limitation

The sole purpose of this workflow is to keep the researcher list in iMarina
in sync with ICIQ's HR records, so ICIQ can report which research staff are
currently active. The A3 dump and the resulting iMarina upload must not be
used, forwarded or retained for any other purpose by anyone with access to
the `runtime/*` SharePoint folders this pipeline reads and writes.

## Data minimization and access restriction

Triggering the workflow is restricted to a small, named set of people with a
legitimate need to do so — see
[How to request an iMarina load](../how-to/request-an-imarina-load.md) for
exactly who. The raw A3 dumps themselves are provided manually, at most once
a month, by ICIQ's Human Resources department, the only authorized source of
that HR extract entering the pipeline at all.

Restricting *who may trigger the workflow* through the form only has real
GDPR value if it's backed by matching SharePoint item-level permissions on
the `_Projects/imarina-load-researchers/runtime/*` folders, and on the
Microsoft List and Form themselves. Restricting the form's submit button is
meaningless if the underlying files stay readable by a broader SharePoint
audience than the people authorized to trigger a run.

This is also why every sharing link this pipeline generates (for the A3
dump, the iMarina upload, the review copy) is created with
**organization-scoped**, view-only access — never anonymous — even though an
anonymous link would be simpler to implement.

## Storage limitation — a known open gap

The "use the last file" fallback that both `download` (for missing input
links) and `publish` (for archiving) rely on depends on every previous A3
dump and iMarina upload remaining in `runtime/a3` and `runtime/published`
indefinitely. As things stand, there's no retention or purge policy for
these historical personal-data files, which conflicts with GDPR's storage
limitation principle.

This is an acknowledged, unresolved gap rather than a design decision: a
retention period should eventually be defined, along with a decision on
whether older dumps can be safely deleted without breaking the "pick the
latest" mechanism the pipeline depends on throughout (see
[Why filenames encode dates](naming-conventions.md)).
