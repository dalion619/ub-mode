---
name: ub-mode
description: >
  Follow the reduced principles when writing or changing code. Every change
  follows First Make it Work, then Then Make it Right, then Follow Practice.
  Gates the result with crapper (CRAP < 10), dryer (similarity < 0.8), and
  mutator (no survivors), C# first. Use when the user runs /ub-mode or asks
  for the reduced principles.
---

# ub-mode

Every time you are asked to make a change follow:
* First Make it Work.
* Then Make it Right.
* Follow Practice

# First Make it Work

## Laziness Protocol

* One decision lives in one place.
* Ship the smallest edit that solves the problem.

## Foundational Thinking

* Get the data shape right before you write the logic.
* Do first whatever helps every later phase.
* If a concurrent writer would matter, do not share state.

## TDD

* Write the failing test before you change the code.
* See it fail for the reason you expect.
* Make the smallest change that makes it pass.
* If an honest test is impractical, do not write a bad one.

## Redesign From First Principles

* Do not bolt a new requirement onto the old design.
* Ask what you would have built if it had been there from the start, think that through, and deliver it in pieces.

## Attack the Premise

* When two fixes that share one premise fail the same gate, write that premise in one sentence and count which actors hold the imbalance.
* Remove the assignment.
* Do not start another fix until you have done that.

## Subtract Before You Add

* Take complexity out before you add anything.

## Outcome-Oriented Execution

* Aim at the end state you can check.
* A break in the middle is allowed when you planned it and can undo it.

## Build the Lever

* If the work is not obvious at a glance, build the tool that does it.
* Do one unit by hand, then make the tool match that unit.

## Architecture

### Model the Domain

* Put the domain in a structure that makes the illegal states impossible.
* Maintain a glossary of domain terms.
* If the code is already clear and local, leave it.

### Boundary Discipline

* Check the data where it enters.
* Inside, trust it.
* Keep the business logic in a pure function.

### Type System Discipline

* Make the illegal value impossible to write.
* Parse outside data once, where it comes in.
* Do not silence the compiler.

### Make Operations Idempotent

* Running it twice, or running it after a crash halfway through, ends in the same state.

### Separate Before Serializing Shared State

* If they do not need the same mutable thing, give each its own.
* If they do, allow one writer, and enforce that in the structure.

# Then Make it Right

## Fix Root Causes

* Reproduce it.
* Fix the cause.
* A guard that only hides the crash is not a fix.

## Sequence work into verifiable units

* Finish one small unit and check it.
* Then start the next.

## Test Behavior, Not Implementation

* The test calls the code the way its user does and checks the result that user would see.
* If the test would still pass when every function returned nothing, throw it out.

## Quality Gates

Three tools gate every change. They write `.metrics/` at the project root, which [uml-viewer](https://github.com/unclebob/uml-viewer) reads. Run them from the project root (the folder with the `.sln` / `.slnx` for C#), in this order: crapper, then dryer, then mutator.

### Setup

* The tools live side by side in `$UB_TOOLS`. If it is unset, `export UB_TOOLS="$HOME/.ub-mode/tools"`. mutator imports crapper from `../crapper`, so keep them siblings.
* If a tool is missing, clone it: `git clone https://github.com/dalion619/crapper`, `.../dryer`, `.../mutator` into that folder.
* Each launcher (`crapper/crapper`, `dryer/dryer`, `mutator/mutator`) builds its own `.venv` on first run. Needs `python3` and `git`.
* Add `.metrics/` and `target/` to `.gitignore`.

### C# Setup

* Needs the .NET SDK on PATH. Check with `dotnet --list-sdks` before running anything.
* Every test project needs `coverlet.collector`. Without it coverage is 0% and every CRAP score is a lie. `dotnet add <TestProject> package coverlet.collector`.
* Name test projects `*.Tests`, `*.Test`, `*.UnitTests`, `*.IntegrationTests`, or `*.Specs`. Those are skipped as source. Anything else gets scored and mutated as production code.
* `*.g.cs` and `*.Designer.cs` are skipped. Do not hand-edit them to pass a gate.
* Tag tests that must not run under mutation `[Trait("Category", "no-mutate")]`. mutator runs `dotnet test --filter Category!=no-mutate`.
* `dotnet test` must pass before you start. A red baseline makes mutator exit `2` and write nothing.

## Crapper

* Run `$UB_TOOLS/crapper/crapper --threshold 9.99`. Scope it with a path, a fragment, or `--changed`.
* It runs `dotnet test --collect "XPlat Code Coverage"` and writes `target/coverage/csharp/lcov.info` plus `.metrics/crap.edn`.
* Exit `2` means a function scored above 9.99, so it is not under 10. Reduce each one under 10 by splitting it, by testing it, or both.
* C# complexity follows ReSharper: `if`, loops, `catch`, `when` filters, `?:`, each `case` (not `default`), each switch-expression arm (not `_`), `&&`, `||`, `??`, `?.`, and `and` / `or` patterns. A chain of `?.` is complexity. Parse once at the boundary instead.
* Consider whether case statements are worth reducing. A flat switch over a closed enum that is fully tested is fine at CC 12; untested it is not.
* Accessors are entries (`Total.get`). Lambdas and local functions count against their enclosing member.

## Dryer

* Run `$UB_TOOLS/dryer/dryer --threshold 0.8`. Scope it with a path or `--changed`.
* Eliminate every pair at 0.8 or above. Below `--min-lines 4` it does not report; that is fine.
* In C# it compares methods, constructors, finalizers, operators, and accessors. Locals, literals, and field names normalize away. Two methods that differ only in names are the same method; extract it.
* It skips test projects. Duplicated test setup is not a dryer finding.

## Mutator

* Run crapper first, then `$UB_TOOLS/mutator/mutator --use-existing-coverage <file or folder>`. That reuses crapper's coverage instead of running the suite twice.
* Start with the files you changed. The first full run executes every covered mutant and takes as long as the suite times the sites.
* Exit `0` all killed. Exit `2` baseline failed. Exit `3` a mutant survived.
* Cover the uncovered: an `UNCOVERED` line has no test. Write one through the public API.
* Kill the survivors: a `SURVIVED` mutant (`>` to `>=`, `&&` to `||`, `?.` to `.`, `true` to `false`, `0` to `1`) means no test pins that decision. Add the boundary case.
* Delete dead code: a survivor no honest test can kill is code nothing needs. Remove it.
* Reruns are differential. Only survivors and changed members run again. `--mutate-all` forces everything.
* Workers build into their own `target/dotnet-artifacts`. Use `--max-workers` if the machine runs out of memory.

## Refactoring

* Pin what the code does.
* Change the structure.
* Prove the behavior did not change.

# Practice

## Keep a Log

* Keep a log of every action you take and the reason you took it.
* Report that log upon completion.

## Guard the Context Window

* Keep the main thread small.
* Send the bulk elsewhere, and keep the summary.

## Worktree Cleanup

* Find what is safe to remove, confirm nothing still in use needs it, then reclaim the disk.
* Uncommitted work waits for a decision.

## Commit and Push

* Commit often, in small ordered commits.
* Each commit says why, what changed, and how you proved it.
* Push to the main line at the end.
