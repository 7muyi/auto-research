# Experiment lifecycle

Follow this lifecycle for every Auto-research run. Resume from verified Git and filesystem state after an interruption; do not repeat a transition merely because a previous narration or note is missing.

## Repository layout

```text
<repository>/
|-- <existing code and configuration>
|-- goal.md
|-- records.yaml
|-- records/
|   `-- <record-id>/
|       |-- record.md
|       |-- result.md
|       `-- artifacts/
`-- worktrees-<record-id>/
```

The `worktrees-<record-id>` directory is temporary. The Experiment Branch remains after the worktree is removed.

## 1. Initialize the Goal Branch

1. Confirm that the working directory belongs to the intended Git repository. Inspect the current branch, worktree state, existing `goal/main` ref, registered worktrees, and uncommitted changes.
2. Create and switch the primary checkout to `goal/main` from the user-intended starting commit. If this cannot be done safely, report the exact obstacle and wait for the user. Do not select another base, force a checkout, discard changes, or rewrite history.
3. If a compatible Auto-research initialization already exists, validate it and resume instead of recreating it.
4. Otherwise:
   - render `goal.md` from [goal.md](goal.md) and complete every section with the user;
   - initialize `records.yaml` to `{}`;
   - create `records/` and add `records/.gitkeep` so Git can track the empty directory;
   - keep `worktrees-*` out of ordinary commits, preferably through the repository-local Git exclude file if needed.
5. Stage only `goal.md`, `records.yaml`, and `records/.gitkeep`. Inspect the staged paths and diff.
6. Create the exact commit:

   ```text
   init auto-research
   ```

This commit is the initial `goal/main` baseline. The first Record ID is `001`.

## 2. Select an experiment origin

Read the authoritative `goal.md`, `records.yaml`, and prior Records from `goal/main`.

For a **New Research Direction**:

- set `parents: []`;
- use the current `goal/main` commit as `base_commit`;
- describe why the proposed line of investigation is meaningfully non-iterative.

For an **Iterative Experiment**:

- list every finalized Record whose results or conclusions materially inform the new experiment;
- choose exactly one listed Parent Record as the Git base;
- set `base_commit` to that Parent Record's original `record_commit`;
- explain the base selection in `record.md`;
- when multiple parents exist, carry conclusions or implementation from non-base parents through explicit new Code Commits rather than an implicit merge.

Parent edges represent research derivation, not necessarily Git ancestry. Reject a missing parent, duplicate Record ID, or cycle.

## 3. Allocate the Record and create its worktree

1. Find the largest Record ID that has already been used or reserved, increment it, and format the result with at least three digits. Never reuse an ID.
2. Choose a concise lowercase ASCII branch slug containing only letters, digits, and hyphens.
3. Create `feat/<record-id>-<branch-slug>` from the exact `base_commit` and add its linked worktree at the repository-relative path `./worktrees-<record-id>`.
4. Verify that the branch, base commit, registered worktree, and resolved absolute path are exactly the intended ones.
5. In the Experiment Worktree, create:

   ```text
   records/<record-id>/record.md
   records/<record-id>/artifacts/
   ```

6. Render `record.md` from [record.md](record.md). Before experimentation, complete the provenance, hypothesis, motivation, planned changes, and base-selection rationale.

Do not commit the preliminary Record yet. Commit the complete Record exactly once at the end of the experiment.

## 4. Execute and evaluate the experiment

Work only inside the Experiment Worktree for this round.

1. Implement the hypothesis in ordinary code or configuration paths, never inside `records/<record-id>/`.
2. Generate code, repair defects, tune parameters, run experiments, and repeat affected checks until the hypothesis has a valid result, the experiment becomes impossible to complete normally, or a Blocked Stop Condition applies.
3. Capture exact commands, working directories, dependency and data versions, seeds, exit codes, metrics, and other conditions needed to interpret or reproduce the result.
4. Store suitable raw evidence under `records/<record-id>/artifacts/`. Keep summaries in Markdown and raw outputs in artifacts. Exclude secrets, credentials, and sensitive environment values.
5. If code changes after a verification run, rerun every check affected by the change.
6. Render [result.md](result.md). Use `FAILED` only when a required condition, resource, or execution capability is unavailable and the current experiment therefore cannot be completed and must be abandoned. For example, an experiment that requires CUDA may be `FAILED` when no suitable CUDA device is available. Do not use `FAILED` for a completed experiment whose results are negative or below expectations. State whether the Research Goal was achieved and provide evidence for every applicable Acceptance Criterion.
7. Update the Results and Interpretation/Conclusion sections of `record.md` to reflect what actually happened, including material deviations from the plan.

A successful command does not by itself prove that the Research Goal is achieved. Missing required evidence means `Research Goal achieved: NO`.

## 5. Commit experiment code

1. Stage code and runnable configuration with explicit pathspecs.
2. Inspect every staged path and the staged diff. Exclude:
   - `goal.md`;
   - `records.yaml`;
   - every path under `records/`;
   - worktree bookkeeping;
   - unrelated user files.
3. Create one or more Code Commits. Intermediate Code Commits are allowed.
4. Record the full commit hashes in chronological order. The last hash is the code state against which the final result must be verified.

Every experiment must have at least one Code Commit. For a deliberate no-change control or a failed attempt that changed no code, create a documented intentional empty Code Commit.

## 6. Close the Experiment Record

1. Pin `record.md` and `result.md` to the last Code Commit.
2. Confirm that:
   - the execution status is `COMPLETED` or `FAILED`, where `FAILED` means that the experiment could not be completed normally and must be abandoned, not that its completed result was unfavorable;
   - `Research Goal achieved` is `YES` or `NO`;
   - `YES` is used only when the experiment is `COMPLETED` and every applicable Acceptance Criterion passes;
   - the conclusion is supported by retained evidence;
   - the evidence inventory is complete and contains no secrets.
3. Stage only `records/<record-id>/`, including deliberately retained artifacts.
4. Inspect the staged paths and diff. The commit must touch nothing outside the current Record directory.
5. Create exactly one Record Commit and save its full hash. This terminal commit closes the Record.

Do not amend a closed and synchronized Record. Correct later understanding through a new Record or an explicit metadata-only correction on `goal/main` without rewriting prior evidence.

## 7. Synchronize the Record to `goal/main`

1. Return to the primary checkout and verify that it is on `goal/main` with no conflicting changes in the affected paths.
2. Cherry-pick the original Record Commit onto `goal/main`. Use Git's provenance option when available so the generated commit message records the source hash.
3. Verify that the cherry-picked commit changes only `records/<record-id>/` and that the synchronized content matches the Experiment Branch.

The cherry-pick creates a new commit hash. Preserve the original Experiment Branch hash as `records.yaml.git.record_commit`.

## 8. Update the Experiment Index

1. Add exactly one entry to `records.yaml` using [records.yaml](records.yaml).
2. Use the quoted Record ID as the YAML key.
3. Store:
   - the complete ordered `parents` list;
   - a concise description;
   - the execution status only;
   - the full Experiment Branch name;
   - the exact full `base_commit` hash;
   - the ordered list of full Code Commit hashes;
   - the original full Record Commit hash.
4. Validate that the YAML has no duplicate keys, every parent exists, the graph is acyclic, the hashes resolve to the expected commits, and commit path scopes obey the separation rules.
5. Stage and commit only `records.yaml` as a separate `goal/main` commit.

Do not treat an edited but uncommitted Experiment Index as a finalized transition.

## 9. Remove the Experiment Worktree

Remove the worktree only after the Record has been synchronized and the Experiment Index entry has been committed.

1. Inspect tracked, untracked, and ignored files in the Experiment Worktree. Preserve every intended code change and evidence item in the appropriate commit.
2. Resolve the exact absolute target and verify that it:
   - is the registered worktree for this Record;
   - equals the intended `<repository>/worktrees-<record-id>` path;
   - is not the primary checkout.
3. From outside that worktree, remove it with Git's worktree removal operation.
4. Verify that both the registration and directory are gone. Prune only stale registration when needed.
5. Retain `feat/<record-id>-<branch-slug>` for provenance and possible future iteration.

If unknown data remains or ordinary worktree removal fails, preserve the directory and ask the user. Do not substitute unchecked recursive deletion or force removal.

## 10. Decide the next transition

Read the finalized `result.md` after cleanup.

### Research Goal achieved: `NO`

- If no Blocked Stop Condition applies, return to step 2 and start a new Record.
- Use an Iterative Experiment when the next hypothesis inherits prior results.
- Use a New Research Direction when it inherits from no prior Record; start from `goal/main` and set `parents: []`.
- Never reopen the closed Record or reuse its ID.

If a Blocked Stop Condition applies, preserve all finalized state and report the exact condition and supporting evidence. Do not claim success.

### Research Goal achieved: `YES`

1. Determine the complete code lineage required to reproduce the accepted state. Follow the selected Git-base chain by matching each iterative Record's `base_commit` to its chosen Parent Record's `record_commit`.
2. Collect all missing Code Commits in ancestor-first order, ending with the accepted Record's Code Commits. Do not cherry-pick Record Commits or Experiment Index commits as code.
3. On `goal/main`, cherry-pick the required Code Commits in order. If all required ancestor code is already present, cherry-pick only the missing commits.
4. If a conflict occurs, preserve and inspect the in-progress state. Do not blindly discard user work or improvise an unrecorded behavioral change. A new research change requires a new Record.
5. Run every Acceptance Criterion again on the integrated `goal/main` tree. Verification performed only in the deleted Experiment Worktree is insufficient.
6. Preserve integration evidence in the accepted Record through a new metadata-only commit on `goal/main` if the evidence is not already present. Do not rewrite the original Record Commit.
7. Declare the Research Goal complete only when integrated verification passes and all required metadata, evidence, and cleanup are complete.

If integrated verification fails, preserve the proven state and begin a new Experiment unless a Blocked Stop Condition requires stopping.
