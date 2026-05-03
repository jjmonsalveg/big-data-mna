# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

Coursework for "Análisis de grandes volúmenes de datos" (Big Data Analysis), 3rd trimester at ITESM. Contains PySpark Jupyter notebooks per week (`sem1/`, `sem2/`, ...) and a multi-stage course project under `proyecto/` (`etapa1/`, etc.).

## Environment

- Python managed via pyenv; the active env is `env-pyspark` (see `.python-version`).
- PySpark is installed inside that pyenv virtualenv at `~/.pyenv/versions/env-pyspark/lib/python3.12/site-packages/pyspark`.
- Notebooks bootstrap Spark with `findspark.init()` then build a local SparkSession (`SparkSession.builder.master("local[*]")`). No cluster — everything runs locally.

## Common commands

- Activate the env: `pyenv activate env-pyspark` (or rely on `.python-version` auto-activation).
- Launch notebooks: `jupyter lab` (or `jupyter notebook`) from the repo root.
- Data files for notebooks live next to them (e.g. `sem2/classroom/cars.csv`); use relative paths from the notebook directory.

## Conventions

- One notebook per topic/lesson; keep new work inside the appropriate `semN/` folder, or under `proyecto/etapaN/` for project deliverables.
- PDFs in `proyecto/etapaN/` are assignment briefs from the instructor; read them for requirements, do not modify.
- `.idea/`, `Session.vim`, `.ipynb_checkpoints/`, and `.python-version` are gitignored; do not commit IDE/editor state.
- Project deliverables must be portable: classmates and Google Colab should be able to open a notebook and run it. Keep dataset paths relative to the notebook (e.g. `Path("data/raw")` next to the notebook, not at repo root).
- **Never execute the notebook's data-loading or compute cells from the agent side** (no `wget`, `curl`, `jupyter nbconvert --execute`, etc. on dataset downloads or Spark jobs). The user runs cells in Jupyter; the agent only writes them.

## Output style (notebooks, prose, reports)

- No emojis anywhere in user-facing output: notebooks, markdown, prose, print statements, log messages. Decorative checkmarks, arrows, and similar Unicode glyphs are an obvious AI tell — avoid them.
- No em-dashes (—) or en-dashes (–) in prose. Use commas, colons, parentheses, or split into two sentences. Standard hyphens in code identifiers are fine.
- No decorative ASCII separators (`===`, `---`, `>>>`, `-->`) in print output. A blank line is enough.
