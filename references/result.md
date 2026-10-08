# Result: <record-id>

## Outcome

- Execution status: <COMPLETED | FAILED>
- Research Goal achieved: <YES | NO>
- Recommended next action: <ACCEPT | ITERATE | BLOCKED>
- Final code commit: <full Git hash>
- Ordered code commits:
  - <full Git hash>
- Goal commit: <full goal/main commit containing the applicable goal.md>

Use `FAILED` only when the experiment cannot be completed normally and must be abandoned because a required condition or resource is unavailable, or another blocking execution condition makes completion impossible. Do not use `FAILED` for a completed experiment with an unfavorable result.

## Verification

<Describe what was verified, how it was verified, what was observed, where the supporting evidence is stored, and whether the evidence satisfies the applicable Acceptance Criteria. Include commands, environment details, metrics, and exit statuses only when relevant. Use prose, lists, tables, or another clear structure appropriate to the experiment.>

## Reproduction

<Record the environment, dependencies, data versions, seeds, runnable configuration, and sufficient steps to reproduce the result. Keep runtime inputs in normal code or configuration paths; artifacts are retained evidence copies.>

## Analysis

<Analyze the measurements, uncertainty, limitations, regressions, unexpected behavior, and comparison with the baseline or Parent Records.>

## Conclusion

<Justify the execution status, the Research Goal assessment, and the recommended next action. For `FAILED`, identify the unavailable requirement or blocking execution condition that forced abandonment of the experiment. `ACCEPT` requires every applicable Acceptance Criterion to pass. A completed experiment with a negative or non-improving result is `COMPLETED`, normally with `ITERATE` or `BLOCKED`, not `FAILED`.>

## Evidence inventory

<List the raw logs, metrics, configuration snapshots, plots, and other retained evidence under artifacts/. Explain any evidence that is unavailable or intentionally not retained.>
