# arch · design record

**arch** is a desktop tool for Rust codebases, built as an Eclipse Theia product, in which a person and coding agents work on a repository through its architecture map. arch computes the map from rust-analyzer facts, keeps its own notes under `.arch/`, writes nothing into source, and lets you plan a change, delegate it to a session, follow it, review it and merge it, with the manual path always present. The full statement is `spec/arch-v1-spec.md` §1.

This repo is **the record**: the one place where the product's decisions live, with a git history. Every other repo links to it; none depends on it at build time.

## Rules

- A decision is taken in the product chat and written here as an ADR **before** any repo implements it. An implementation session that needs a decision not in `adr/` asks the product chat; it does not decide alone and it does not leave the question in code comments.
- The spec is amended, not forked: when an ADR changes a section, the section is rewritten and the ADR is referenced.
- The flow pages are the visual spec. A page is superseded by moving it to `flows/superseded/`, never by editing it in place; a new page is a new file.
- Plain text, no build step, no generated content. Diagrams are the HTML pages or Mermaid in Markdown.

## Repo map

```
arch-design/
  README.md                    this page
  roadmap.md                   V1 · V2 · V3 (spec §3), kept current
  spec/
    arch-v1-spec.md            the V1 specification; §13 holds the decisions of 2 Oct 2026
    vocabulary.md              the terms, with the C4 mapping
    keyboard.md                the keyboard map
    map-invariants.md          MAP-1…32 and CANVAS-1…2 as testable statements, each with its owner and check
  flows/                       the HTML pages, one file each, self-contained (open in a browser)
    shell.html                 the shell: one bar, rail, left panel, bottom panel, status bar
    plan-shell.html            plan and delegate, on the shell
    session-shell.html         session: follow · live · asks · deviation · gate · take over · done
    review-merge-shell.html    review and merge: glance · detail · remark · verdict · merge · merged
    daily-shell.html           daily: map and selection · read code · Ask · edit by hand · finding · What's new · commit
    map-focus.html             the map: anatomy, folding, navigation, overlays, edit at scale, invariants
    map-real-projects.html     the map on real projects (Zed, tokio, Bevy, Vector)
    v2-canvas.html             V2 canvas: zoom levels, tiers, sockets → ports, flow overlay
    superseded/                the earlier pages, kept for content not redrawn (see spec §2)
  adr/                         one file per decision: context · decision · consequences, numbered, dated
  handoffs/                    every handoff written so far, and the per-repo ones
```

## ADRs

Synced-to line for the other repos: **ADR 0029** (2026-10-03). Later decisions are added as 0030, 0031, …

| # | decision |
|---|---|
| [0001](adr/0001-store-sqlite.md) | Store: sqlite in `.arch/cache/`, plain text committed |
| [0002](adr/0002-keys-commit-hash-deltas.md) | Keys: facts by commit hash as per-file deltas |
| [0003](adr/0003-threads-and-records.md) | Threads and records: on the session's branch, archived to git notes at merge |
| [0004](adr/0004-areas-derived-no-source-annotation.md) | Areas: derived from the module tree, overrides only, no annotation in source |
| [0005](adr/0005-context-pack.md) | Context pack: the standard input to planner, sub-agents, judge and Ask |
| [0006](adr/0006-arch-init.md) | `arch init`: one commit, no questions |
| [0007](adr/0007-unit-in-v1.md) | Unit in V1: the main `[[bin]]` or the lib, and its closure |
| [0008](adr/0008-non-rust-ignored.md) | Non-Rust: ignored, except table names from `migrations/*.sql` |
| [0009](adr/0009-confidence-levels.md) | Confidence: `resolved` · `guessed` · `declared` |
| [0010](adr/0010-degrade.md) | Degrade: syntax-level facts marked guessed when rust-analyzer cannot type-check |
| [0011](adr/0011-worktrees.md) | Worktrees: `<parent>/<repo>-w<n>` |
| [0012](adr/0012-confinement-driver-hooks.md) | Confinement: the driver's pre-edit and post-edit hooks |
| [0013](adr/0013-judge.md) | Judge: same provider, fresh invocation, fixed prompt, structured verdict |
| [0014](adr/0014-concurrent-sessions.md) | Concurrent sessions: warn, show overlaps, never lock |
| [0015](adr/0015-restart.md) | Restart: resume each running session at its last turn boundary |
| [0016](adr/0016-commits.md) | Commits: agents commit through arch's tool, one per element, with trailers |
| [0017](adr/0017-push.md) | Push: agents never push |
| [0018](adr/0018-forge-github-first.md) | Forge: GitHub first, GitLab in V1.x, `Forge` trait from day one |
| [0019](adr/0019-conflicts-resolve-element.md) | Conflicts: a resolve element on the session's plan, fast gate |
| [0020](adr/0020-plan-drafts.md) | Plan drafts: in the cache until Accept; discard deletes |
| [0021](adr/0021-element-identity.md) | Element identity: content-addressed, `E3` as label, continuity by site then similarity |
| [0022](adr/0022-review-without-plan.md) | Review without a plan: works for any branch |
| [0023](adr/0023-settings-errors-secrets-packaging-telemetry.md) | Settings, errors, secrets, packaging, telemetry |
| [0024](adr/0024-repositories.md) | Repositories: arch · arch-app · sett · arch-fixtures · arch-design |
| [0025](adr/0025-direction-call-family-grain.md) | Direction: only call-family links carry it, with the grain of the column rule (left → right in layers, inward in hexagon) |
| [0026](adr/0026-budgets-first-index-memory.md) | Budgets: first index < 5 s cold at 1,200 items; `arch` < 500 MB RSS at 5,000 visible items; one benchmark per budget |
| [0027](adr/0027-arch-init-sides-rule.md) | `arch init` sides: entry → driving, I/O external → driven, otherwise domain; split one level on a driving/driven mix; override in `areas.toml` |
| [0028](adr/0028-entry-kinds-spawned-worker.md) | Entry kinds: a function spawned once at start-up that loops for the life of the process is an entry (`spawned worker`); `resolved` when spawned in `main`, `guessed` otherwise |
| [0029](adr/0029-arch-init-order-rule.md) | `arch init` order: alphabetical by area name within a side (bytes, ascending); one area per top-level module, a single file included; grouping is a hand override; after `init` the file is the order |
## Flow pages

| page | what it is |
|---|---|
| [shell](flows/shell.html) | the retained shell and its invariants SHELL-1…4, LEFT-0…8 |
| [plan](flows/plan-shell.html) · [session](flows/session-shell.html) · [review and merge](flows/review-merge-shell.html) · [daily](flows/daily-shell.html) | the four flows on the shell (spec §6) |
| [map focus](flows/map-focus.html) · [real projects](flows/map-real-projects.html) · [v2 canvas](flows/v2-canvas.html) | the map (spec §5) |
| `flows/superseded/` | ask · base · findings · firsts · git · merge · onboarding · plan · review · session · shell-flows · sync; still cited by spec §2 for the Update dialog, the fix card, judgement rows and reconcile |

## Handoffs

| file | for |
|---|---|
| [handoff-arch-design.md](handoffs/handoff-arch-design.md) | this repo |
| [handoff-arch.md](handoffs/handoff-arch.md) | `tau-rs/arch`, the engine |
| [handoff-arch-app.md](handoffs/handoff-arch-app.md) | `tau-rs/arch-app`, the Theia product |
| [handoff-arch-fixtures.md](handoffs/handoff-arch-fixtures.md) | `tau-rs/arch-fixtures`, test data and invariants |
| [handoff-sett-rebase.md](handoffs/handoff-sett-rebase.md) | `tau-rs/sett`, the design system: rebase on the retained shell |
| [handoff-design-system.md](handoffs/handoff-design-system.md) | the design-system side-work (done; sett exists) |
| [handoff-plugin-system.md](handoffs/handoff-plugin-system.md) · [handoff-plugin-system-spec.md](handoffs/handoff-plugin-system-spec.md) | the plugin system side-work (designed; V2) |
| [handoff-visualizer-poc.md](handoffs/handoff-visualizer-poc.md) | the visualizer proof of concept (done) |

## The other repos

| repo | is | states |
|---|---|---|
| [`tau-rs/arch`](https://github.com/tau-rs/arch) | the engine: facts, views, server, tool layer, drivers, forge; one Cargo workspace | the ADR it is synced to |
| [`tau-rs/arch-app`](https://github.com/tau-rs/arch-app) | the Theia desktop product, consuming `@tau-rs/sett` | the ADR it is synced to |
| [`tau-rs/sett`](https://github.com/tau-rs/sett) | the design system: tokens, Lit `sett-*` elements, Storybook; `DESIGN.md` wins on look, the ADRs on behaviour and vocabulary | the ADR it is synced to |
| [`tau-rs/arch-fixtures`](https://github.com/tau-rs/arch-fixtures) | pinned real repos, golden facts, invariant and performance checks | the ADR it is synced to |

## Sync with the other repos

Weekly, or at every milestone of a repo: an implementation session reads `adr/` newer than its last sync and `roadmap.md`, applies what changed, and files its own findings (a surface the shell lacks, a fact the analyzer can't produce, a component sett refuses) as issues here labelled `from:<repo>`. The product chat triages those into ADRs.
