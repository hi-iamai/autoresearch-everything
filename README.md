# autoresearch-everything

`autoresearch-everything` is a Codex skill for turning a research goal, raw context, and constraints into a runnable autoresearch workspace.

It uses [karpathy/autoresearch](https://github.com/karpathy/autoresearch) as the default execution substrate, then generates the surrounding research organization: `program.md`, `research_state.md`, lightweight support skills, and optional subagent briefs.

## What It Does

- Clone or update `karpathy/autoresearch`.
- Convert a goal plus messy notes/logs/examples into an explicit research contract.
- Generate or refine `program.md` for an autonomous experiment loop.
- Scaffold `research_state.md`, metric parsing guidance, and a critic subagent brief.
- Preserve the central autoresearch pattern: propose a small experiment, run an objective check, keep improvements, discard regressions, and log what happened.

The included `2026-06-17_09-00-55Z-(5)-filter.md` is an example of a highly specialized investigation transcript. The skill treats examples like that as raw evidence to mine for structure, not as templates to copy.

## Repository Layout

```text
autoresearch-everything/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   └── program-design.md
└── scripts/
    └── bootstrap_autoresearch.py
```

Key files:

- `autoresearch-everything/SKILL.md`: the skill entrypoint and workflow.
- `autoresearch-everything/references/program-design.md`: detailed guidance for designing `program.md`, support skills, and subagent briefs.
- `autoresearch-everything/scripts/bootstrap_autoresearch.py`: repeatable bootstrap script for creating an autoresearch workspace.

## Quick Start

Create or update an autoresearch workspace:

```bash
python autoresearch-everything/scripts/bootstrap_autoresearch.py \
  --workspace ./runs/my-topic \
  --write-program \
  --goal "Improve validation bits-per-byte under the fixed time budget" \
  --metric "val_bpb, lower is better" \
  --experiment-command "uv run train.py" \
  --validation-command "uv run train.py"
```

This creates:

```text
runs/my-topic/
├── upstream/              # karpathy/autoresearch checkout
├── research_state.md
├── skills/
│   └── metric-reader.md
└── subagents/
    └── critic.md
```

When `--write-program` is present, the script writes `runs/my-topic/upstream/program.md`. If an upstream `program.md` already exists, it is backed up once as `program.upstream.md`.

Use `--no-pull` when offline or when you want to preserve the existing `upstream` checkout exactly.

## Script Options

```text
--workspace              Workspace directory to create or update.
--no-pull                Do not pull an existing upstream checkout.
--write-program          Write a generated upstream/program.md scaffold.
--goal                   Research mission to place in program.md.
--metric                 Authoritative metric and direction.
--experiment-command     Command that runs one experiment.
--validation-command     Command that validates or parses the metric.
--editable               Editable path; may be repeated.
--forbidden              Forbidden/read-only path; may be repeated.
```

Example for a non-ML codebase:

```bash
python autoresearch-everything/scripts/bootstrap_autoresearch.py \
  --workspace ./runs/latency \
  --write-program \
  --goal "Reduce p95 request latency while preserving API behavior" \
  --metric "p95 latency, lower is better" \
  --experiment-command "pytest -q" \
  --validation-command "pytest -q" \
  --editable src \
  --forbidden tests/fixtures
```

## Using the Skill in Codex

Invoke it with a prompt like:

```text
Use $autoresearch-everything to turn this goal, context, logs, and constraints into a runnable autoresearch setup.
```

Good inputs include:

- the goal and success metric
- baseline logs or benchmark output
- files agents may edit
- files agents must not edit
- setup and validation commands
- prior research notes or chat transcripts
- compute, time, and risk limits

If the metric is subjective, define a review rubric rather than pretending it is numeric.

## Validation

Validate the skill structure with:

```bash
python C:\Users\vyeli\.codex\skills\.system\skill-creator\scripts\quick_validate.py autoresearch-everything
```

Check the bootstrap script with:

```bash
python -m py_compile autoresearch-everything/scripts/bootstrap_autoresearch.py
python autoresearch-everything/scripts/bootstrap_autoresearch.py --help
```

## Notes

- Network access is required the first time the bootstrap script clones `karpathy/autoresearch`.
- The generated `program.md` is a starting point. Refine it with the guidance in `references/program-design.md`.
- Keep generated support skills short; put long evidence and run history in `research_state.md` or attempt logs.
