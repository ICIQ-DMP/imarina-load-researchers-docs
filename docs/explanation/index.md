# Explanation

Explanations are **understanding-oriented**. They clarify and illuminate
the design decisions, architecture, and concepts behind imarina-load-researchers.

---

- [Architecture overview](architecture-overview.md) — why the pipeline is
  five independent commands talking only through a filesystem convention.
- [How `build` matches researchers between A3 and iMarina](how-build-matches-researchers.md)
  — the update algorithm behind each generated file.
- [The end-to-end workflow](the-end-to-end-workflow.md) — the full,
  human-and-system journey from a submitted request to a published file.
- [Data protection and GDPR](data-protection.md) — purpose limitation, data
  minimization and the storage-limitation gap this design still has.
- [Secrets and credentials](secrets-and-credentials.md) — why credentials
  are read through one cascading function instead of per-environment code.
- [Why filenames encode dates](naming-conventions.md) — the "latest file"
  convention this pipeline relies on throughout, and its one exception.
