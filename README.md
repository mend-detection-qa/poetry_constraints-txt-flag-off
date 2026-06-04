# poetry_constraints-txt-flag-off

**What this probe proves:** When `applyConstraints=false`, Mend ignores `constraints.txt` entirely on a Poetry project even though the file is present at the repo root. The gate is the only thing controlling the behavior.

## Pairing with poetry_constraints-txt-overlay

This probe uses **identical Python project content** to `poetry_constraints-txt-overlay` — same `pyproject.toml`, same `poetry.lock`, same `constraints.txt`. The only difference is `whitesource.config`:

| Probe | `python.applyConstraints` | Expected `urllib3` |
|---|---|---|
| `poetry_constraints-txt-overlay`  | `true`  | `1.26.18` (from constraints.txt) |
| `poetry_constraints-txt-flag-off` (this) | `false` | `2.7.0` (poetry.lock unconstrained value) |

Without this negative half, a passing `-overlay` probe could mean "Mend applied the constraint" OR "poetry.lock happened to land on 1.26.18 anyway". Together the pair gives a load-bearing proof the gate works.

## Real-world scenario

A customer wants to roll out the feature gradually. They have `constraints.txt` files in many Poetry-based repos but only want Mend to honor them in some. Per-repo opt-in via `whitesource.config` is how they control it.

## Files

- `pyproject.toml` — Poetry manifest pulling `requests = "2.32.0"` (identical to `poetry_constraints-txt-overlay`)
- `poetry.lock` — **NOT YET GENERATED.** Run `poetry lock` on a real machine. The lock will reflect Poetry's unconstrained resolution (urllib3 at latest 2.x). Do NOT hand-craft.
- `constraints.txt` — `urllib3==1.26.18` (identical content; Mend should NOT read it)
- `whitesource.config` — `python.applyConstraints=false`

## Expected scan behavior (post SCA-5154)

Dep tree mirrors the unconstrained Poetry resolution from `poetry.lock`:

- Direct: `requests` at `2.32.0`
- Transitive `urllib3` at the latest 2.x release available at lock time (NOT the constrained 1.26.18)
- All other transitives at their resolver defaults

## Load-bearing assertion

`expected_dependency_pairs` pins `urllib3@2.7.0`. If Mend silently honored the constraint despite the flag being off, urllib3 would land at 1.26.18 → assertion fails.