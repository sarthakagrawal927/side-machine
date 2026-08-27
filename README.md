# Side Machine

Current release: `v1.0.0`. See [versioning and releases](docs/VERSIONING.md)
and the [changelog](CHANGELOG.md).

Side Machine is a minimal, zero-incremental-cost pattern for offloading
persistent Claude Code and Codex CLI sessions from a primary Mac to a spare Mac
on the same local network.

It uses native macOS Remote Login, OpenSSH, Herdr, caffeinate, and independent
Git checkouts. tmux remains installed as a recovery fallback. It is not a
custom control plane.

## Status

Worker setup and health tooling validated locally; controller helper and
dedicated-key setup are ready to install.

Read [the product requirements document](docs/PRD.md) before setup or
implementation.

## Intended experience

```bash
side-chick
side-chick alive
side-chick status
side-chick speed
side-chick watch
side-chick herdr
side-chick version
```

`side-chick` opens a normal SSH shell. The alive command checks SSH without
authenticating. The status dashboard shows CPU, memory, disk, power, Wi-Fi link
rate and signal, persistent services, Herdr sessions, and agent processes.
`side-chick speed` runs Apple's `networkQuality` for real upload/download
throughput. Watch mode retries after outages and uses passwordless SSH to avoid
repeated prompts. Run `side-chick setup-key` once to create and authorize a
dedicated key; never store the account password in the command or a dotfile.
Herdr owns persistent agent terminals; direct SSH remains the recovery path.

## Local helper installation

On the worker:

```bash
install -m 755 bin/side-machine-status ~/.local/bin/side-machine-status
install -m 755 bin/side-machine-speed ~/.local/bin/side-machine-speed
```

On the controller, copy and install the helper:

```bash
mkdir -p ~/.local/bin
scp assistant@Assistants-MacBook-Pro.local:~/Desktop/side-machine/bin/side-chick ~/.local/bin/side-chick
chmod 755 ~/.local/bin/side-chick
```

Ensure `~/.local/bin` is on the controller's `PATH`, then remove any older
`side-chick` alias so the command is used.

## Project boundary

In scope:

- LAN-only controller-to-worker access;
- persistent Claude Code and Codex terminal sessions;
- passwordless SSH, Herdr persistence, and worker status;
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
