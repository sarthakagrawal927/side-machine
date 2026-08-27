# Changelog

Side Machine uses semantic version tags. See `docs/VERSIONING.md` for the
release and migration policy.

## [2.0.0] — planned

- Use the existing Tailscale `fleet-mac` node as the default transport.
- Remove the same-Wi-Fi requirement while retaining LAN fallback.
- Surface Tailscale reachability and transport state in health output.

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
