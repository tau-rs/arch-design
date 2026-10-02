# ADR 0023 · Settings, errors, secrets, packaging, telemetry

- Date: 2026-10-02
- Status: accepted
- Source: `spec/arch-v1-spec.md` §13.23 (decisions of 2 Oct 2026, taken in the product chat)

## Context

Cross-cutting decisions the shell and the engine need before the first screen: where settings live, how errors surface under P-2, where secrets go, how the app and engine ship, and whether anything is reported home.

## Decision

- **Settings**: a Settings tab (this repo · you · appearance).
- **Errors**: a status-bar state per subsystem plus a Checks row; **never a modal**; the last good view is kept.
- **Secrets**: the OS keychain via Theia; **never in `.arch`**.
- **Packaging**: a Theia desktop app spawning **`arch`** as a separate binary; the `arch` CLI is **`init · serve · check · mcp · hook`**.
- **Telemetry**: **none**.

## Consequences

- `arch-app` and `arch` are separate repos and processes ([ADR 0024](0024-repositories.md)); the API schema is their contract.
- Degraded analysis ([ADR 0010](0010-degrade.md)) and forge failures surface through the same status-bar and Checks path.
- `arch check` is the CI entry; `arch mcp` serves agents; `arch hook` is what the driver's hooks call ([ADR 0012](0012-confinement-driver-hooks.md)).
