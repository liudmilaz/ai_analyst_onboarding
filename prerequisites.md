# Prerequisites

Written for someone starting from nothing. **You do not need any of this to
begin** — Phases 1 and 2 are analysis and research, and need only a text editor
and your agent. Phase 3 is where software gets installed.

## Phases 1–2 — nothing to install

An agent that reads `AGENTS.md`, and the CSVs in `data/`. That is all.

## Phase 3 onward — a container runtime, and nothing else

The project asks for the **entire stack** to be containerized. You install a
container runtime; everything else — database, dbt, BI platform — comes up
inside it. You do **not** install a database, dbt, Python or a BI tool on your
machine. That is the whole point of the constraint: the deliverable has to work
for someone who has none of your local setup.

| Requirement | Check | If missing |
|---|---|---|
| Docker CLI | `docker --version` | `brew install docker` (macOS) or your package manager |
| Compose | `docker compose version` | `brew install docker-compose` |
| A container runtime | `docker info` | See below |

That is the complete list.

### Choosing a runtime

Docker Desktop is the obvious choice but **requires a paid licence for business
use**, which conflicts with the open-source-only constraint in
`requirements.md`. Open alternatives include Colima, Podman and Rancher Desktop.

If you use **Colima on Apple Silicon**, name the VM type explicitly:

```bash
colima start --vm-type vz --cpu 4 --memory 8 --disk 60
```

Without `--vm-type vz` it falls back to QEMU, and if QEMU is not installed it
**hangs silently** rather than reporting an error. The VM also does not survive
a reboot, so expect to start it at the beginning of a session.

### dbt: Core, in a container

Use **dbt Core**, the open-source command-line tool. dbt Cloud is a hosted SaaS
product and is ruled out by the local-deployment constraint.

**Run it as a service, not from a virtual environment.** Official images exist
per adapter (`dbt-postgres`, `dbt-duckdb` and so on), or you can build a small
image from `python:slim`. Either way it joins the compose network and reaches
the database by its **service name** — not `localhost`, which inside a container
means the container itself.

The temptation is to `pip install dbt-postgres` locally because it is two
minutes faster. Resist it: a host-installed dependency is exactly the kind of
step that works on your machine and fails on the next person's, which is what
the constraint exists to prevent.

## Agent capabilities

The workshop leans on a handful of capabilities. Any agent that has them works;
none of them require a specific product.

| Capability | Used in | If your agent lacks it |
|---|---|---|
| Visible task/progress tracking | All phases | Keep a `PROGRESS.md` and update it |
| Plan-before-acting | Phases 2–6 | Write the plan in chat and agree it explicitly |
| Structured brainstorming | Phases 1–2 | Work the trade-offs in conversation |
| Test-first discipline | Phase 4 | Write the test file before the model file. It is a habit, not a feature |
| Systematic debugging | Phase 3 onward | Reproduce, one hypothesis, test it, then change something |

Some agents package these as installable extensions — Claude Code has a plugin
marketplace, for instance. **If you install one, verify the commands actually
exist before relying on them.** A missing extension usually fails silently: the
command does nothing, no error appears, and it is entirely possible to finish
the workshop believing you used tooling you never invoked.

## Version control

The repository already has git history. Work on a copy so the original stays
available to compare against:

```bash
cp -r . ../workspace && cd ../workspace
```

Commit as you go — per-phase commits are enough. There is no need to run
`git init`, and no need to have your agent generate an instructions file: this
repository ships one.
