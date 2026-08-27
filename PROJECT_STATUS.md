# Side Machine — PROJECT STATUS

Last updated: 2026-08-27

Current implementation version: `v2.0.2`

## Why / What

Provide a polished, minimal way to offload persistent Claude Code and Codex CLI
sessions from a primary Mac to a spare Mac over their private tailnet or LAN.

**Users:** Mac owners with a second machine and existing Claude/Codex access.

**IN scope:** native SSH over Tailscale or LAN, Herdr persistence,
agent-scoped sleep prevention, status commands, optional Cursor Remote SSH,
and Git worktree isolation.

**OUT of scope:** public SSH, router forwarding, containers, custom
orchestration, local model inference, and reboot-transparent sessions.

## Dependencies

### External

- macOS Remote Login and OpenSSH
- Tailscale on the controller and the existing Fleet-managed worker node
- Herdr, with tmux retained as a recovery fallback
- Claude Code and an eligible account
- Codex CLI and an eligible account

### Internal

- Shared Fleet operating standard in `../AGENTS.md`

## Timeline

- 2026-08-26 — standalone PRD-first project created.
- 2026-08-27 — v1 commands and v2 private Tailscale transport published.

## Products

- Minimal Remote Mac Agent Worker setup
- Controller-side status and session helpers

## Features (shipped)

- Complete implementation PRD with phase gates, acceptance tests, rollback,
  and agent handoff prompt.
- Worker health dashboard with CPU, memory, disk, power, Wi-Fi link quality,
  Herdr sessions, agent counts, and service state.
- Controller `side-chick` helper for shell, reachability, status, watch, actual
  network speed tests, Herdr attach, and dedicated-key setup.
- Persistent launchd services for Herdr and caffeinate validated on the worker.
- Documented M5-to-M1 and iPhone-to-either-Mac Tailscale connectivity model.

## Todo / Planned / Deferred / Blocked

1. Update the M5 controller helper to the current v2 patch.
2. Verify `side-chick status` from the M5 after the worker-helper fix.
3. Reconnect `iphone171` and verify phone-to-M1 SSH.
4. Enable and verify restricted Remote Login on the M5 for phone access.
5. Prove Claude Code and Codex Herdr detach/reattach flows end to end.
6. Validate worker reachability after a real controller sleep/network outage.
7. Select project visibility and license before publishing.
8. Create CI only when explicitly approved.
