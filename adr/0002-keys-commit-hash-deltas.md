# ADR 0002 · Keys: facts by commit hash as per-file deltas

- Date: 2026-10-02
- Status: accepted
- Source: `spec/arch-v1-spec.md` §13.2 (decisions of 2 Oct 2026, taken in the product chat)

## Context

Facts must be cheap to recompute for one changed file (< 500 ms, spec §5) and must survive branches, worktrees, uncommitted edits and rebases without a full rebuild. Commits are facts in the V1 data model (spec §3).

## Decision

- Facts are keyed by **commit hash**, stored as **per-file deltas** with content hashes.
- Branches are pointers to a commit's facts.
- Uncommitted work is keyed by a **worktree-state hash**.
- A rebase reuses deltas by file hash.

## Consequences

- A worktree's facts are its base commit's facts plus the deltas of the files that differ; the watcher recomputes only changed files (spec §6 Sync).
- Two sessions on the same base share facts; concurrent-session overlap ([ADR 0014](0014-concurrent-sessions.md)) is computed from the deltas.
- The schema in `arch-facts` ([ADR 0001](0001-store-sqlite.md)) carries commit, file hash and worktree-state hash as keys from day one.
