# How to add a new CLI option

This guide is for contributors adding a new option to one of the five
subcommands (`download`, `build`, `upload`, `publish`, `notify`). The
codebase follows one consistent pattern for this — follow it rather than
calling `typer.Option(...)` inline, both because ruff's `B008` rule flags
most inline calls, and for consistency with every existing option.

## The pattern

Don't write this in a `commands/*/cli.py` controller:

```python
# Don't do this
def build_controller(
    some_flag: Path = typer.Option(Path("default/value"), help="..."),
) -> None:
    ...
```

Instead, split the option into two pieces:

### 1. Metadata, in `core/shared_options.py`

Add an `Annotated` alias with the CLI metadata only — help text and any
flag-name overrides — but **no default value**:

```python
SomeOpt = Annotated[Path, typer.Option(help="What this option does.")]
```

### 2. The default, in `core/defines.py`

If the default is a trivial literal (e.g. `False`, `None`, a short string),
it can stay inline in the controller signature. Otherwise, add a
`DEFAULT_*` constant next to whatever it's derived from:

```python
# next to LOCAL_INPUT_DIR, NOW, REQUIRED_INPUT_FILES, etc. as appropriate
DEFAULT_SOME = LOCAL_INPUT_DIR / "some-file.xlsx"
```

### 3. Wire both into the controller

```python
from imarina_load_researchers.core.defines import DEFAULT_SOME
from imarina_load_researchers.core.shared_options import SomeOpt

def build_controller(
    ...,
    some_param: SomeOpt = DEFAULT_SOME,
) -> None:
    ...
```

## Required options

For an option with **no default** (required), skip the default entirely:

```python
some_param: SomeOpt  # no `= ...`
```

Don't use Typer's `typer.Option(...)` (Ellipsis-means-required) idiom here —
it type-checks fine at runtime, but this project runs `mypy --strict`, which
has no special case for a bare `...` default against a non-Optional
annotation and will report a spurious error. Python requires no-default
parameters to come before ones that have a default, so just omit the `=`
clause instead.

## Sharing an option across commands

If more than one command needs the exact same option, define it once in
`shared_options.py` and import it into each controller — see `IdOpt`
(shared by `build`/`upload`/`publish`) as the existing example.

## Verify it

After wiring the option in, confirm it shows up correctly:

```shell
./venv/bin/imarina-load-researchers build --help
```

...and run the full check suite before opening a PR:

```shell
make lint   # ruff + mypy
make test   # pytest
```
