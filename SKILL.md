---
name: auto-research
description: Automatically conduct goal-driven scientific research by exploring, experimenting with, and iteratively improving solutions until the research objectives are achieved.
disable-model-invocation: true
---

# Auto-research

Turn an open-ended research task into a traceable sequence of experiments. Form hypotheses, implement and evaluate them, retain evidence, learn from prior results, and continue until the Research Goal is achieved or a declared Blocked Stop Condition applies.

Use Git as the versioned state machine for the research process. Each **Experiment Record**, shortened to **Record**, is one independently closed experiment and one node in the Research Graph. 

NOTE: (Parent Record edges express research provenance; Git ancestry expresses the selected implementation base. Do not conflate them.)

Before changing a repository, read [the experiment lifecycle](references/workflow.md). Use these templates when creating research files:

- [Research Goal contract](references/goal.md) for `goal.md`.
- [Experiment Record](references/record.md) for `records/<record-id>/record.md`.
- [Experiment result](references/result.md) for `records/<record-id>/result.md`.
- [Experiment Index](references/records.yaml) for `records.yaml`.

## Research contract

- Keep `goal.md`, `records.yaml`, and `records/` at the repository root. Preserve the repository's existing code layout.
- Treat `goal.md` as the static Research Goal contract. It defines the objective, Acceptance Criteria, constraints, authorized boundaries, and Blocked Stop Conditions. Do not change it during research unless the user explicitly authorizes a contract revision.
- Use `goal/main` as the Goal Branch. It is authoritative for the Research Goal, Experiment Index, and synchronized Records.
- Use `feat/record-<record-id>-<branch-slug>` as an Experiment Branch and `./worktrees-<record-id>` as its temporary Experiment Worktree.
- Allocate globally unique, monotonically increasing Record IDs, starting at `001`. Format them as zero-padded decimal strings and never reuse an ID, including after a failed experiment.
- A **New Research Direction** is a non-iterative line of investigation that inherits from no prior Record. Create it from `goal/main` and set `parents: []`.
- An **Iterative Experiment** extends prior research. List every Record whose conclusions materially informed it in `parents`, select one Parent Record's `record_commit` as the Git `base_commit`, and document the base choice. Multiple research parents do not imply a Git merge.
- Keep the Research Graph acyclic. Every Parent Record must already be finalized in `records.yaml`.

## Separation and finality

- Keep runnable code and research metadata separate. Code must not depend on `goal.md`, `records.yaml`, or `records/` at runtime.
- Store only research information and evidence in `records/<record-id>/`. Put raw logs, metrics, configuration snapshots, plots, and other reproducibility evidence in its `artifacts/` directory. Do not store secrets.
- Every Record has at least one Code Commit followed by exactly one terminal Record Commit. A Code Commit must exclude `goal.md`, `records.yaml`, and all `records/` paths. The Record Commit must include only `records/<record-id>/`.
- If a valid experiment makes no code changes, create one intentional empty Code Commit and explain why. Do not use an empty commit to hide uncommitted work.
- The Record Commit closes the experiment. Do not amend or reopen a synchronized Record; start a new Record for further experimentation.
- `records.yaml.git.code_commits` is the ordered list of full hashes for Code Commits created during that Record. `record_commit` is the original full hash of the Record Commit on the Experiment Branch, not the hash created by cherry-picking it onto `goal/main`.

## Outcome semantics

Keep execution status separate from Research Goal attainment:

- `COMPLETED` means the experiment finished with a valid, interpretable result. A negative or non-improving result can still be `COMPLETED`.
- `FAILED` means the experiment could not be completed normally and this Record must be abandoned because a required condition or resource was unavailable, or another blocking execution condition made completion impossible—for example, required CUDA hardware was unavailable.
- `FAILED` abandons the current experiment only. Continue with a different experiment when possible; stop the overall research only when the cause also satisfies a Blocked Stop Condition in `goal.md`.
- `result.md` must explicitly state `Research Goal achieved: YES` or `NO` and justify it criterion by criterion. `YES` requires a `COMPLETED` experiment and passing evidence for every applicable Acceptance Criterion.
- If the Research Goal is not achieved and no Blocked Stop Condition applies, close the current Record and begin another experiment. If a Blocked Stop Condition applies, preserve the evidence and stop with a precise report.

## Operational safeguards

- Inspect repository status, branches, registered worktrees, and relevant paths before mutation. Preserve unrelated user changes; do not stash, reset, overwrite, or commit them without authorization.
- If `goal/main` cannot be created or checked out safely, stop and tell the user. Do not choose a substitute branch or rewrite history.
- Before every commit, inspect the staged path list and staged diff to enforce code/metadata separation.
- Synchronize the Record and commit its `records.yaml` entry before removing the Experiment Worktree.
- Remove only the verified registered path for the current Experiment Worktree. Inspect tracked, untracked, and ignored files first. If safe removal fails or unknown data remains, preserve it and ask the user rather than forcing deletion.
- Remove the worktree checkout and registration, but retain the Experiment Branch for provenance.
- Integrate accepted code only after the Record is synchronized, the Experiment Index is committed, and the Experiment Worktree is removed. Re-run the Acceptance Criteria on the integrated `goal/main` state before declaring success.
