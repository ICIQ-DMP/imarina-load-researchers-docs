# How-To Guides

How-to guides are **task-oriented**. They show you how to solve a specific
real-world problem, assuming you already know the basics.

---

- [How to request an iMarina load](request-an-imarina-load.md) — for
  requesters: fill in the form, review the generated file, approve or
  reject publication.
- [How to run the full pipeline locally](run-the-pipeline-locally.md) —
  reproduce a real `download` → `build` → `upload` → `notify` run by hand.
- [How to configure secrets](configure-secrets.md) — supply SharePoint, FTP
  and SMTP credentials via a local file, environment variables, Docker
  secrets or Vault.
- [How to override build's input files](override-build-inputs.md) — point
  `build` at files anywhere on disk instead of `./input/`.
- [How to publish a load file manually](publish-a-load-file-manually.md) —
  run the human-gated FTP push directly, safely (dry-run by default).
- [How to add a new CLI option](add-a-cli-option.md) — for contributors:
  the project's consistent pattern for adding options to any subcommand.
