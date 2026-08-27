# Changelog

Side Machine uses semantic version tags. See `docs/VERSIONING.md` for the
release and migration policy.

## [2.0.2] — 2026-08-27

- Document the M5 controller, M1 worker, and iPhone Tailscale topology.
- Add a connection-readiness matrix and phone-to-either-Mac setup.
- Clarify that Tailscale supplies networking while phone terminal access still
  requires SSH and an SSH client.

## [2.0.1] — 2026-08-27

- Install worker helpers as real files so Tailscale SSH does not cross the
  macOS Desktop privacy boundary through a symlink.
- Automatically accept a new SSH host key on first use while continuing to
  reject changed host keys.
- Add SSH keepalives for connections that cross changing networks.

## [2.0.0] — 2026-08-27

- Use the existing Tailscale `fleet-mac` node as the default transport.
- Remove the same-Wi-Fi requirement while retaining explicit LAN fallback.
- Surface Tailscale reachability and transport state in health output.
- Keep the v1 command surface stable and add `lan` and `transport` commands.

## [1.0.0] — 2026-08-27

- Add the controller-side `side-chick` command.
- Add `alive`, `status`, `speed`, `watch`, `herdr`, and `setup-key` commands.
- Add the visual worker dashboard with CPU, memory, disk, battery, Wi-Fi link,
  agent, Herdr, and caffeinate state.
- Add Apple `networkQuality` upload/download testing.
- Replace tmux with Herdr as the primary persistent multiplexer; retain tmux
  as a recovery fallback.
- Validate persistent launchd services for Herdr and caffeinate.

## [0.0.0] — 2026-08-26

- Original PRD-only baseline.
- Define the LAN-only SSH, tmux, caffeinate, agent, safety, and acceptance
  requirements before implementation.
