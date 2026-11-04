# Getting started

This tutorial walks you through setting up `imarina-load-researchers` on your
machine and running the `build` command end to end, against a small,
self-contained sample dataset — no SharePoint, Jenkins or credentials
required. By the end you'll have generated a real iMarina load spreadsheet
and understand the shape of the 9 files `build` needs.

## What you'll need

- Linux (these steps use `apt`; adapt the package names for your distro).
- About 10 minutes.

## 1. Install the prerequisites

`imarina-load-researchers` targets Python 3.13. On Ubuntu:

```shell
sudo apt install python3.13-venv gcc build-essential git -y
```

## 2. Clone the repository and install

```shell
git clone https://github.com/ICIQ-DMP/imarina-load-researchers.git
cd imarina-load-researchers
make install
```

`make install` creates a virtualenv at `venv/` and installs the
`imarina-load-researchers` console script into it. Confirm it worked:

```shell
./venv/bin/imarina-load-researchers --help
```

You should see five subcommands: `download`, `build`, `upload`, `publish`
and `notify`. This tutorial only uses `build` — the one command that needs no
network access or credentials, since it only transforms local files.

## 3. Create a sample dataset

`build` reads 9 fixed-name `.xlsx` files from an `input/` folder: the A3 HR
dump, the previous iMarina upload, and 7 small translation dictionaries (see
[Architecture overview](../explanation/architecture-overview.md) for how
these fit into the pipeline). In production these come from `download`, but for
this tutorial we'll generate a minimal, valid set directly, so you can see
the whole pipeline work without needing real HR data or SharePoint access.

Create a file named `make_sample_inputs.py` with the following content:

```python
"""Generates a minimal, valid set of the 9 input files `build` expects."""

from pathlib import Path

import pandas as pd

out = Path("input")
out.mkdir(parents=True, exist_ok=True)

# --- A3.xlsx: skiprows=2, header=0 -> two throwaway rows, then the header ---
a3_columns = [
    "Código Centro", "Nombre trabajador", "Primer apellido trabajador",
    "Segundo apellido trabajador", "NIF", "Sexo", "Nacionalidad",
    "Pais nacimiento _", "E-mail profesional", "Puesto de trabajo",
    "Grupo Unidad", "ORCID", "Fecha Inicio Contrato", "Fecha Fin Contrato",
    "Fecha Inicio Prórroga", "Fecha Fin Prórroga", "Fecha de baja en compañía",
]
a3_row = [
    1, "Ada", "Lovelace", "", "12345678A", "D", "España", "España",
    "ada.lovelace@example.com", "Investigador", "QOC", "0000-0002-1234-5678",
    "01/01/2024", "", "", "", "",
]
with pd.ExcelWriter(out / "A3.xlsx") as writer:
    pd.DataFrame([a3_row], columns=a3_columns).to_excel(
        writer, index=False, header=True, startrow=2
    )

# --- iMarina.xlsx: header=0, zero data rows (first-ever load) ---
imarina_columns = [
    "nombre", "primer_apellido", "segundo_apellido", "signature",
    "signature_custom", "DNI/NIE/NIF", "Fecha de Nacimiento", "Sexo",
    "País de Nacimiento", "Correo Electrónico", "Web Personal",
    "Tipo de Adscripción", "Categoría Investigadora/Docente", "Dedicación",
    "Fecha de Inicio", "Fecha de Fin", "Entidad (Nivel 1)", "Tipo de Entidad",
    "País de la Entidad", "Region/Comunidad de la Entidad",
    "Provincia de la Entidad", "Ciudad de la Entidad",
    "Código Postal de la Entidad", "Dirección de la Entidad",
    "Web de la Entidad", "ORCID", "Google Scholar ID", "AuthorID (Scopus)",
    "Teléfono de Contacto",
]
pd.DataFrame(columns=imarina_columns).to_excel(out / "iMarina.xlsx", index=False)


# --- Two-column translation dictionaries: skiprows=1, header=None ---
def write_dict(filename, title, rows):
    with pd.ExcelWriter(out / filename) as writer:
        pd.DataFrame([[title, ""], *[list(r) for r in rows]]).to_excel(
            writer, index=False, header=False
        )


write_dict(
    "Pais nacimiento _ [A3] to País de Nacimiento [iMarina].xlsx",
    "A3 -> iMarina country", [("España", "Spain")],
)
write_dict(
    "Puesto de trabajo [A3] to Categoría Investigadora Docente [iMarina].xlsx",
    "A3 -> iMarina job description", [("Investigador", "Researcher")],
)
write_dict(
    "Grupo Unidad [A3] to Web personal [iMarina].xlsx",
    "Entity -> personal web", [("ICIQ", "https://iciq.org/")],
)
write_dict(
    "Grupo Unidad [A3] to Entidad (Nivel 1) [iMarina].xlsx",
    "A3 unit group -> iMarina entity", [("QOC", "ICIQ")],
)
write_dict(
    "Entidad (Nivel 1) [iMarina] to Tipo de Entidad [iMarina].xlsx",
    "entity -> entity type", [("ICIQ", "Research Institute")],
)
write_dict(
    "Puesto de trabajo [A3] to Entidad (Nivel 1) [iMarina].xlsx",
    "special job descriptions -> entity",
    [("__unused__", "__unused__")],  # no row in this file is entirely empty
)
write_dict(
    "Sexo [A3] to Sexo [iMarina].xlsx",
    "A3 -> iMarina sex", [("H", "Male"), ("D", "Female")],
)

print("Wrote sample input files to", out.resolve())
```

Run it:

```shell
./venv/bin/python make_sample_inputs.py
```

This writes the 9 files into `./input/`, describing one researcher — Ada
Lovelace, a female researcher in the "QOC" unit — plus an empty previous
iMarina upload (as if this were ICIQ's very first load).

## 4. Run `build`

```shell
mkdir -p output
./venv/bin/imarina-load-researchers build
```

You should see log output ending in something like:

```text
Since the last upload, 0 researchers have changed its position within ICIQ.
Since the last upload, 0 researchers have visited ICIQ.
Since the last upload, 0 researchers have left ICIQ.
Since the last upload, 1 researchers have entered ICIQ.
iMarina Excel at output/2026-08-26_11-16-04__icl_ag_personal_12539.xlsx built successfully.
```

## 5. Inspect the result

Open the generated file in `output/` (the filename starts with the datetime
`build` ran at). You'll find one row, with the A3 data translated into
iMarina's format — Ada's unit group `QOC` became the entity `ICIQ`, her
country `España` became `Spain`, her job title `Investigador` became
`Researcher`, and her personal web page was looked up from the entity's
dictionary entry.

That translation — matching each A3 column against a small dictionary file,
then filling a row in the iMarina spreadsheet's own columns — is the whole
job of `build`. Re-run it with this same generated file as the new
`input/iMarina.xlsx`, and a fresh (identical) `input/A3.xlsx`, and you'll see
`build` recognize that Ada is unchanged, rather than reporting her as a new
hire again.

## Next steps

- [How to run the full pipeline locally](../how-to/run-the-pipeline-locally.md)
  shows how `download`, `upload` and `publish` fit around `build` in a real
  run.
- [How `build` matches researchers between A3 and iMarina](../explanation/how-build-matches-researchers.md)
  explains the update algorithm behind the "0 changed" / "1 entered" lines
  you just saw in the logs.
- [Architecture overview](../explanation/architecture-overview.md) explains
  *why* the pipeline is shaped this way — five independent commands,
  talking only through fixed filenames.
- [How to override build's input files](../how-to/override-build-inputs.md)
  covers pointing individual `--*-dict`/`--a3-input`/`--imarina-input`
  options at files anywhere on disk, instead of `./input/`.
