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
- Verify every external URL with WebFetch before writing it into a notebook, report, or any deliverable. Do not paste URLs from memory or training data: API doc paths change between versions and broken references damage the deliverable's credibility.
- Spanish prose and Spanish comments **must** use proper accents and `ñ`. Writing "seccion" instead of "sección", "metodo" instead of "método", "anos" instead of "años" (this one is especially bad — `año` and `ano` mean very different things), "mas" instead of "más", "automatico" instead of "automático", "tambien" instead of "también", "estan" instead of "están", or "dia" instead of "día" is **incorrect Spanish**, not a stylistic choice. It is an obvious AI tell and unacceptable in academic deliverables. This rule applies in: markdown cells, print/log strings, code comments, column aliases, and any other Spanish text. The deliverable language for this project is Spanish, so this rule is load-bearing.
- **Reason this is enforced strictly**: the user pays per token to delegate work to subagents. Producing Spanish without accents forces a re-processing pass to fix grammar, which is a pure cost waste. Subagents writing Spanish must verify the prose has accents BEFORE returning. If unsure about a specific word, look it up; do not guess.
- **Never reference internal agent names in user-facing deliverable prose** (notebooks, reports, comments seen by graders). Phrases like "el grupo `data-quality` decidirá", "el agente `eda-analyst`", "`viz-specialist` generará el gráfico" are internal orchestration metadata, not academic content. In deliverables, refer to upcoming work neutrally: "la etapa de calidad", "el análisis exploratorio", "la visualización", "el equipo del proyecto", "se evaluará en pasos posteriores". Agent names belong in the orchestrator's internal logs and in `.claude/agents/*.md`, never in the student's prose.
