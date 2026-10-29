# ub-mode

`ub-mode` is an agent skill that enforces disciplined software engineering principles when writing or changing code. It is loosely based on portions of [@poteto](https://github.com/poteto)'s rules (`poteto-mode` / `pstack`), reduced and paired with deterministic quality-gate tools:

1. **First Make it Work** — Laziness Protocol, Foundational Thinking, TDD, Redesign From First Principles, Attack the Premise, Subtract Before You Add, Outcome-Oriented Execution, Build the Lever, and Architecture (Model the Domain, Boundary Discipline, Type System Discipline, Make Operations Idempotent, Separate Before Serializing Shared State).
2. **Then Make it Right** — Fix Root Causes, Sequence Work into Verifiable Units, Test Behavior (Not Implementation), [Crapper](https://github.com/dalion619/crapper) (`CRAP < 10`), [Dryer](https://github.com/dalion619/dryer) (duplication `< 0.8`), [Mutator](https://github.com/dalion619/mutator) (cover the uncovered, kill the survivors, delete dead code), and Refactoring.
3. **Practice** — Keep a Log, Guard the Context Window, Worktree Cleanup, and Commit and Push in small ordered commits.

## Companion Tool Repositories

`ub-mode` relies on deterministic quality-gate tools to enforce structure, complexity, duplication, and test thoroughness:

- [dalion619/crapper](https://github.com/dalion619/crapper) — Computes cyclomatic complexity, test coverage, and CRAP score per function so every function stays under the CRAP threshold (`< 10`).
- [dalion619/dryer](https://github.com/dalion619/dryer) — Detects structural code duplication so duplicate blocks (`>= 0.8` similarity) are eliminated.
- [dalion619/mutator](https://github.com/dalion619/mutator) — Runs mutation testing to expose uncovered code and surviving mutants that tests fail to catch.
- [unclebob/uml-viewer](https://github.com/unclebob/uml-viewer) — Visualizes components, coverage, CRAP, mutation scores, and dependency directions.

All three cover Clojure, Java, C#, Go, TypeScript, Rust, and Python. The skill is written C# first: coverage comes from coverlet through `dotnet test` (or Microsoft.Testing.Platform's own coverage when `global.json` opts into it), complexity follows ReSharper's cyclomatic rules, and test projects are recognised by name (`*.Tests`, `*.UnitTests`, ...).

## Installation

Copy `SKILL.md` into your agent's skills directory as `ub-mode/SKILL.md` (for example `~/.grok/skills/ub-mode/SKILL.md`, `~/.claude/skills/ub-mode/SKILL.md`, or `~/.cursor/skills/ub-mode/SKILL.md`), then invoke `/ub-mode`.

Clone the tools side by side (mutator imports crapper from `../crapper`):

```bash
mkdir -p ~/.ub-mode/tools && cd ~/.ub-mode/tools
git clone https://github.com/dalion619/crapper
git clone https://github.com/dalion619/dryer
git clone https://github.com/dalion619/mutator
```

Set `UB_TOOLS` if you put them somewhere else. Each launcher builds its own `.venv` on first run.

For C# on VSTest (the default), every test project needs `coverlet.collector`:

```bash
dotnet add tests/MyApp.Tests package coverlet.collector
```

On Microsoft.Testing.Platform (`global.json` has `"test": {"runner": "Microsoft.Testing.Platform"}`), test projects need `Microsoft.Testing.Extensions.CodeCoverage` instead. `MSTest.Sdk` projects already include it.

Then, from the folder with the `.sln`:

```bash
~/.ub-mode/tools/crapper/crapper --threshold 9.99
~/.ub-mode/tools/dryer/dryer --threshold 0.8
~/.ub-mode/tools/mutator/mutator --use-existing-coverage src/MyApp
```
