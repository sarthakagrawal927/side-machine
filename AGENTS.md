# Side Machine agent instructions

Also read and follow the shared Fleet standard at `../AGENTS.md`.

## Product boundary

This project specifies and implements a minimal private-tailnet remote Mac
worker for persistent Claude Code and Codex CLI sessions, with same-LAN SSH as
a fallback. Read `docs/PRD.md` before making material changes.

Keep the active stack limited to native macOS Remote Login, OpenSSH, Tailscale,
Herdr (with tmux retained only as a recovery fallback), caffeinate, Git, Claude
Code, Codex CLI, and optional Cursor Remote SSH. Herdr replaced tmux as the
primary multiplexer and Tailscale became the default transport by owner
decision on 2026-08-27.

Do not introduce Coder, Colima, Docker, Kubernetes, Headscale,
monitoring services, a database, a web control plane, or a custom scheduler
without a new owner decision that changes the PRD.

## Safety

- Never read, print, copy, or commit private SSH keys, agent credentials,
  tokens, environment files, or Keychain contents.
- Ask before creating or authorizing an SSH key, enabling Remote Login, adding
  a persistent LaunchAgent, or changing power, firewall, or router settings.
- Preserve existing SSH and tmux configuration with small, scoped edits.
- Never expose SSH publicly or add router port forwarding under this PRD.
- Preserve dirty repositories and use separate worktrees for concurrent
  writing agents.
- Do not commit, push, publish, deploy, or release unless explicitly asked.

## Verification

Use the PRD acceptance matrix. Run the smallest relevant check first and
report exact receipts without sensitive values.
