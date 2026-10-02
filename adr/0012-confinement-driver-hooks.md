# ADR 0012 · Confinement: the driver's pre-edit and post-edit hooks

- Date: 2026-10-02
- Status: accepted
- Source: `spec/arch-v1-spec.md` §13.12 (decisions of 2 Oct 2026, taken in the product chat)

## Context

Every agent write must pass through the tool layer with the content hash of the last read (spec §7); the only core veto is that an element's sub-agent may only write its element's files (spec §8); writes must be attributed to the sub-agent for the overlays and the `you` detection (spec §4).

## Decision

- The driver's **pre-edit hook** calls arch: the stale-write guard, then the element-scope veto.
- The driver's **post-edit hook** attributes the write.
- This is the **only confinement in V1**.
- Verify on day one that `--settings` hooks fire under `--bare`.

## Consequences

- No sandbox, no file-system jail, no proxy in V1; a driver whose hooks do not fire is not usable, hence the day-one check.
- `arch hook` is a CLI subcommand for this ([ADR 0023](0023-settings-errors-secrets-packaging-telemetry.md)).
- Direct `git commit` is denied through the same layer ([ADR 0016](0016-commits.md)).
- The three seams for V2 plugins (pipeline: stale-write guard → veto → attribution) are kept as spec §8 requires.
