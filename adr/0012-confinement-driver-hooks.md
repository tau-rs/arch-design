# ADR 0012 · Confinement: the driver's pre-edit and post-edit hooks

- Date: 2026-10-02
- Status: accepted
- Source: `spec/arch-v1-spec.md` §13.12 (decisions of 2 Oct 2026, taken in the product chat)
- Settled since: tau-rs/arch-design#15 is decided (2026-10-03): hooks passed with `--settings` never fire under `--bare`, so the claude-code driver does not use `--bare`; it keeps the user's settings out with `--setting-sources '' --strict-mcp-config` instead, and the decision gives the command line
- Settled since: tau-rs/arch-design#33 is decided (2026-10-03): the driver drops `--no-session-persistence` and starts each session with an arch-generated `--session-id <uuid>`; answers, fix rounds and restart continue it with `--resume <uuid>`

## Context

Every agent write must pass through the tool layer with the content hash of the last read (spec §7); the only core veto is that an element's sub-agent may only write its element's files (spec §8); writes must be attributed to the sub-agent for the overlays and the `you` detection (spec §4).

## Decision

- The driver's **pre-edit hook** calls arch: the stale-write guard, then the element-scope veto.
- The driver's **post-edit hook** attributes the write.
- This is the **only confinement in V1**.
- **The claude-code driver does not run under `--bare`.** The day-one check was run (tau-rs/arch#12, `experiments/hooks-under-bare.sh`, Claude Code 2.1.272):

  | run | pre/post-edit hooks fire | an exit-2 hook blocks the tool |
  |---|---|---|
  | `--settings` hooks, no other flag | yes | yes |
  | `--bare` + `--settings` hooks | never | — |
  | `--setting-sources ''` + `--strict-mcp-config` + `--settings` hooks | yes | yes |

  `--bare` also accepts only an API key, never the user's claude.ai login. The adapter invokes, in the worktree:

  ```
  claude -p --output-format stream-json --verbose --include-hook-events
         --setting-sources '' --strict-mcp-config --session-id <uuid>
         --settings <arch hooks + permissions> --mcp-config <arch MCP server>
         --append-system-prompt-file <context pack> --allowedTools …
         --permission-mode acceptEdits --permission-prompts none
  ```

  Why: `--setting-sources ''` plus `--strict-mcp-config` keeps the user's own settings, hooks and MCP servers out, which is what `--bare` was chosen for, while the hooks arch passes with `--settings` still fire and the user's normal login works. One honest consequence: this combination does not switch off CLAUDE.md auto-discovery, LSP, plugin sync or auto-memory; each gets its own switch if a later finding shows it matters. Rejected: keeping `--bare` and sending the hooks over the SDK control protocol (API key only, an untested path, more adapter code).

  arch generates the `<uuid>` and records it, with the transcript path, as the [ADR 0003](0003-threads-and-records.md) pointers. A later turn of the same session (the answer to an Ask, a fix round, the [ADR 0015](0015-restart.md) restart) runs the same command with `--resume <uuid>` in place of `--session-id <uuid>`; Claude Code keeps the session id on resume. Why (tau-rs/arch-design#33): each of these picks the session up again, and `--no-session-persistence` means a session is never saved and cannot be resumed. One honest consequence: arch's runs are stored in Claude Code's own history under each worktree's path (`~/.claude/projects/<worktree>/`), so they appear in that directory's `claude --resume` list. Rejected: keeping `--no-session-persistence` and running every turn cold, re-fed from `thread.jsonl` (the agent loses its working reasoning on answers and fix rounds, ADR 0015 would mean restarting the element, and the transcript pointer would always be empty).

## Consequences

- No sandbox, no file-system jail, no proxy in V1; a driver whose hooks do not fire is not usable, hence the day-one check, which ruled out `--bare`.
- `arch hook` is a CLI subcommand for this ([ADR 0023](0023-settings-errors-secrets-packaging-telemetry.md)).
- Direct `git commit` is denied through the same layer ([ADR 0016](0016-commits.md)).
- The three seams for V2 plugins (pipeline: stale-write guard → veto → attribution) are kept as spec §8 requires.
