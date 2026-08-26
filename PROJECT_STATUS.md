# Side Machine — PROJECT STATUS

Last updated: 2026-08-26

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
- tmux
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

## Todo / Planned / Deferred / Blocked

1. Validate the PRD against the actual worker Mac.
2. Implement dependency-free controller status and session helpers.
3. Add worker tmux configuration and setup checks.
4. Prove Claude Code and Codex detach/reattach flows end to end.
5. Select project visibility and license before publishing.
6. Create the remote repository and CI only when explicitly approved.
