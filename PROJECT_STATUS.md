# Side Machine — PROJECT STATUS

Last updated: 2026-08-27

Current implementation version: `v1.0.0`

## Why / What

Provide a polished, minimal way to offload persistent Claude Code and Codex CLI
sessions from a primary Mac to a spare Mac on the same LAN.

**Users:** Mac owners with a second machine and existing Claude/Codex access.

**IN scope:** native SSH, tmux persistence, agent-scoped sleep prevention,
true-colour terminal behaviour, status commands, optional Cursor Remote SSH,
and Git worktree isolation.

**OUT of scope:** paid infrastructure, off-LAN networking, containers, custom
orchestration, local model inference, and reboot-transparent sessions.

## Dependencies

### External

- macOS Remote Login and OpenSSH
- Herdr, with tmux retained as a recovery fallback
- Claude Code and an eligible account
- Codex CLI and an eligible account

### Internal

- Shared Fleet operating standard in `../AGENTS.md`

## Timeline

- 2026-08-26 — standalone PRD-first project created.

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

## Todo / Planned / Deferred / Blocked

1. Install the updated controller helper and run `side-chick setup-key`.
2. Prove Claude Code and Codex Herdr detach/reattach flows end to end.
3. Validate worker reachability after a real controller sleep/network outage.
4. Select project visibility and license before publishing.
5. Create CI only when explicitly approved.
