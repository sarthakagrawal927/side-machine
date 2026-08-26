# Side Machine

Side Machine is a minimal, zero-incremental-cost pattern for offloading persistent
Claude Code and Codex CLI sessions from a primary Mac to a spare Mac on the
same local network.

It uses native macOS Remote Login, OpenSSH, tmux, caffeinate, and independent
Git checkouts. It is not a custom orchestrator or control plane.

## Status

PRD complete; implementation pending validation on the worker Mac.

Read [the product requirements document](docs/PRD.md) before setup or
implementation.

## Intended experience

```bash
side-chick status
side-chick watch
side-chick run ~/code/example codex
side-chick attach example-codex
```

Direct SSH and tmux commands remain the recovery path at every stage.

## Project boundary

In scope:

- LAN-only controller-to-worker access;
- persistent Claude Code and Codex terminal sessions;
- passwordless SSH, tmux true colour, and worker status;
- small, reversible, agent-executable setup steps.

Out of scope:

- Coder, containers, Kubernetes, Tailscale, Headscale, and public SSH;
- cloud scheduling, queues, dashboards, or synchronized filesystems;
- local model inference;
- hiding worker reboot or MacBook lid-close limitations.

## For setup agents

Use the copyable handoff prompt in the PRD. Start with read-only inventory,
work through the phase gates in order, and do not inspect or expose private
keys, tokens, environment files, or agent credential stores.

## License

No license has been selected yet.
