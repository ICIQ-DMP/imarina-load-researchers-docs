# How to request an iMarina researcher load

This guide is for anyone at ICIQ who needs the researcher list in iMarina
refreshed — you don't need to know anything about the code or the CLI to
follow it.

## Who can do this

Triggering this workflow is restricted to a small, named set of people, for
data-protection reasons (see
[Data protection and GDPR](../explanation/data-protection.md)):

- Dr. Sonia Sayalero, responsible for Institutional Strengthening operations
  at ICIQ.
- Aleix Mariné-Tena, ICIQ's data steward.
- An apprentice, if this project is assigned to one.

If you're not one of these people and you need a load run, ask one of them.

## 1. Fill in the request form

Open the iMarina load request Microsoft Form and submit it. Two fields are
**optional**:

- A link to a fresh A3 HR database export.
- A link to a specific previous iMarina upload to use as the baseline.

If you leave either one blank, the system automatically uses the most recent
matching file it already has on SharePoint — the latest A3 dump under
`runtime/a3`, or the latest published iMarina file under `runtime/published`.
In practice, you only need to supply the A3 link when HR has just handed you
a fresh export that hasn't been uploaded yet; you almost never need to supply
the iMarina link.

If you *are* supplying a fresh A3 export yourself, upload it first to
`_Projects/imarina-load-researchers/runtime/a3` on SharePoint, named
`{DATETIME}__listado_personal_A3.xlsx` (e.g.
`2025-03-12_12-00-00__listado_personal_A3.xlsx` — use `12-00-00` if you don't
know the exact time), then copy a sharing link to it into the form.

## 2. Wait for the automated run

Submitting the form creates a tracked request and kicks off the pipeline
automatically — nothing else for you to do at this stage. See
[The end-to-end workflow](../explanation/the-end-to-end-workflow.md) for what
happens behind the scenes.

## 3. Review the generated file

Once the automated run finishes, you'll receive an email with a link to the
generated file on SharePoint, and a Microsoft Approval request asking whether
it should be published to the live iMarina server. Open the file and check
it looks right — new hires present, people who've left flagged with an end
date, changed positions reflected — before deciding.

## 4. Approve or reject

- **Approve** the Microsoft Approval to publish the file to iMarina. You'll
  get a second email once the publish has actually gone through.
- **Reject** it if something looks wrong. No file is published, and you can
  investigate (or ask HR for a corrected A3 export and submit a new request)
  without anything having reached the live iMarina server.

## If something goes wrong

If any step of the automated run fails, you'll get a failure email instead of
a review request. Forward it to the Digitalization team (`digitalitzacio@iciq.es`)
along with the request ID mentioned in the email.
