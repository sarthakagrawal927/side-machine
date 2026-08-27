# Side Machine PRD

Status: ready for implementation  
Version: 1.0  
Last reviewed: 2026-08-26  
Reference topology: one primary Mac controlling one spare Apple Silicon Mac

## Owner amendment — 2026-08-27

Herdr replaces tmux as the primary persistent terminal and agent workspace
manager. tmux remains installed as a direct recovery fallback. The approved
controller helper now includes worker reachability, a visual health dashboard,
Wi-Fi link and throughput checks, outage-aware watch mode, Herdr attachment,
and dedicated SSH-key setup. All LAN-only, credential-safety, repository
isolation, cost, and no-public-exposure requirements remain unchanged.

## Summary

Create a zero-incremental-cost, LAN-only way to run persistent Claude Code and
Codex CLI sessions on a second Mac. The operator works from a primary Mac while
the worker Mac owns the repository checkout, agent process, builds, tests, and
local services.

The solution is deliberately small:

```text
primary Mac
├── terminal
├── Cursor (optional)
└── native SSH client
        │
        │ local network
        ▼
worker Mac
├── macOS Remote Login (OpenSSH)
├── tmux
├── caffeinate
├── Claude Code
├── Codex CLI
└── independent Git checkouts and worktrees
```

There is no control plane, daemon, scheduler, container platform, public
endpoint, or file-synchronization service.

## Product decision

Use native SSH plus tmux. Do not introduce Coder, Colima, Docker, Kubernetes,
Headscale, Tailscale, a web dashboard, distributed queues, or a custom
orchestrator for this requirement.

This design optimizes for:

- zero new infrastructure subscription;
- terminal-first Claude Code and Codex usage;
- sessions that survive network and terminal disconnections;
- fast reattachment with full colour and interactive behaviour;
- transparent operation that another coding agent can inspect and repair;
- small, reversible changes to both Macs.

The model services themselves are not free infrastructure. The operator must
already have an eligible Claude account and an eligible ChatGPT/Codex account,
or separately choose usage-based API billing.

## Goals

1. Start or resume an agent session from the primary Mac in a few seconds.
2. Keep the session running when SSH or the primary terminal disconnects.
3. Preserve ANSI colour, true-colour output, Unicode, mouse scrolling, and
   normal interactive controls.
4. Show worker health and active sessions through one memorable command.
5. Support Claude Code and Codex without coupling the setup to a repository.
6. Prevent concurrent agents from writing to the same Git checkout.
7. Remain private to the local network with no router port forwarding.
8. Give setup agents explicit checks, stop conditions, and evidence to report.

## Non-goals

- Access from outside the local network.
- Surviving worker reboots without restarting an agent.
- Running model inference locally.
- Mirroring unsaved files between Macs.
- Managing cloud agents, CI runners, or production workloads.
- Centralized secrets, audit logs, quotas, scheduling, or multi-user RBAC.
- Replacing Git, tmux, Cursor Remote SSH, Claude Code, or Codex.
- Creating a reusable software product named `mesh`, `m1`, or similar.

## Terms

- **Controller:** the primary Mac where the operator types, monitors, and opens
  Cursor.
- **Worker:** the spare Mac that runs agents, checkouts, builds, and tests.
- **Worker alias:** the internal controller-side SSH name, `m1-worker`.
- **Worker hostname:** the worker's Bonjour name, such as `side-machine.local`.
- **Operator command:** the friendly controller-side reference, `side-chick`.
- **Session:** a named tmux session containing one agent process.
- **Checkout:** one repository working directory. Concurrent writers must use
  separate checkouts or Git worktrees.

## Experience target

The final command surface should be no larger than this:

```bash
side-chick shell
side-chick status
side-chick watch
side-chick sessions
side-chick attach <session>
side-chick run <remote-directory> <codex|claude> [session-name]
```

The helper is a thin shell wrapper around SSH, tmux, and caffeinate. It must not
become a scheduler, repository catalog, task database, background service, or
configuration framework.

Direct commands remain valid recovery paths:

```bash
ssh m1-worker
ssh m1-worker 'tmux list-sessions'
ssh -t m1-worker 'tmux attach-session -t <session>'
```

## Operator decisions required before setup

The setup agent must record these non-secret values before changing anything:

| Decision | Example | Rule |
| --- | --- | --- |
| Controller label | `primary-mac` | Documentation only |
| Worker computer name | `Side Machine` | Human-readable macOS name |
| Worker local hostname | `side-machine` | Produces `side-machine.local` |
| Worker macOS username | `operator` | Never assume it matches the controller |
| Controller SSH alias | `m1-worker` | Stable internal SSH target |
| Operator command | `side-chick` | Friendly user-facing command |
| Remote repository root | `~/code` | Must already exist or be explicitly created |
| Network scope | same LAN only | Stop if remote internet access is required |
| Lid behaviour | open or supported clamshell | `caffeinate` cannot defeat lid-close sleep |
| Authentication | dedicated SSH key | Requires explicit approval before key creation |

Do not record serial numbers, IP addresses, tokens, credential files, private
keys, account email addresses, or repository credentials in this document or
in setup evidence.

## Functional requirements

### FR1: Read-only inventory first

Each agent must begin on its assigned Mac with a read-only inventory:

- macOS version and architecture;
- computer name, local hostname, and current username;
- memory and available disk space;
- network interface and same-LAN reachability;
- presence and versions of SSH, Git, Homebrew, tmux, Claude Code, Codex, and
  Cursor Remote SSH where relevant;
- existing SSH and tmux configuration that would overlap the change;
- existing running tmux sessions and agent processes.

Do not read or print private SSH keys, agent credential stores, environment
files, tokens, or API keys. Preserve existing configuration and make the
smallest compatible diff.

### FR2: Stable worker identity

The worker must have an agreed computer name and local hostname. The local
hostname should resolve from the controller as `<worker-hostname>.local`.

Prefer a router DHCP reservation if `.local` discovery proves unreliable. Do
not configure a manual static IP on macOS unless the operator specifically
chooses it.

Acceptance evidence:

```bash
scutil --get ComputerName
scutil --get LocalHostName
dscacheutil -q host -a name <worker-hostname>.local
```

### FR3: Native Remote Login

Enable macOS Remote Login on the worker under System Settings, General,
Sharing. Restrict access to the designated worker user. Do not enable full disk
access for remote users unless a concrete workflow requires it and the operator
approves it.

Never forward port 22 from the internet-facing router. This PRD assumes both
Macs are on the same trusted LAN.

The first connection must validate the worker's host fingerprint rather than
blindly accepting an unexpected key.

### FR4: Passwordless authentication

After one successful password-authenticated connection, create a dedicated
Ed25519 key on the controller only if the operator explicitly approves SSH-key
changes.

Requirements:

- use a worker-specific key rather than reusing or copying a private key;
- add only the public key to the worker user's `authorized_keys`;
- include a descriptive public-key comment;
- never display, copy, upload, or commit the private key;
- preserve any existing `authorized_keys` entries;
- verify a second connection succeeds without a password before ending the
  initial connection.

If key creation is not approved, password authentication remains a supported
but lower-quality fallback.

### FR5: Controller SSH configuration

Merge one host block into the controller's existing SSH configuration:

```sshconfig
Host m1-worker
    HostName side-machine.local
    User <worker-user>
    Port 22
    ConnectTimeout 10
    ServerAliveInterval 30
    ServerAliveCountMax 3
    ControlMaster auto
    ControlPath ~/.ssh/control-%C
    ControlPersist 10m
```

Add an `IdentityFile` only when a dedicated key was approved and created. Do
not replace existing includes or wildcard blocks. Validate the effective
configuration with `ssh -G <worker-alias>`.

### FR6: Worker packages

The minimum worker package set is:

- Git, using the macOS/Xcode Command Line Tools installation if already
  available;
- tmux;
- Claude Code;
- Codex CLI.

Homebrew is optional infrastructure for installing tmux. Do not install Docker,
Colima, Coder, Glances, Kubernetes, a prompt framework, or a language-version
manager merely to satisfy this PRD.

Before running an agent vendor's installer, retrieve the current official
instructions and compare them with this PRD. As of the review date, the
official standalone installers are:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
curl -fsSL https://claude.ai/install.sh | bash
```

Do not run a changed domain, mirrored installer, or third-party package. Record
only the resulting version, never installer output containing private account
information.

Verification:

```bash
git --version
tmux -V
codex --version
claude --version
claude doctor
```

### FR7: Agent authentication

Authentication must occur on the worker and remain on the worker.

For Codex on a remote or headless worker, prefer the official device-code flow:

```bash
codex login --device-auth
codex login status
```

If device authentication is unavailable, use the official localhost callback
forwarding flow. Do not copy `~/.codex/auth.json` as the normal setup path. It
contains credentials and must never appear in documentation, tickets, chat, or
setup output.

For Claude Code:

```bash
claude
claude auth status
```

When the browser cannot open from SSH, copy the displayed login URL into the
controller browser and return the one-time code if prompted. On macOS, Claude
Code stores credentials in the login Keychain. Do not add API keys to shell
startup files as part of this setup.

### FR8: Worker power behaviour

Each agent process must prevent idle system and disk sleep only while it is
running:

```bash
/usr/bin/caffeinate -i -m codex
/usr/bin/caffeinate -i -m claude
```

This allows display sleep. It does not prevent shutdown, reboot, battery loss,
or MacBook lid-close sleep. The worker should remain plugged in with the lid
open or use a supported clamshell setup.

A persistent per-user LaunchAgent running `caffeinate -i -m` is optional and
requires a separate explicit operator decision. It is not the default because
it keeps the worker awake even when no agent is active.

Validate an active assertion with:

```bash
pmset -g assertions
```

### FR9: tmux persistence and visual quality

Install or merge these tmux behaviours on the worker:

```tmux
set -g default-terminal "tmux-256color"
set -as terminal-features ",xterm-256color:RGB"
set -g history-limit 100000
set -g mouse on
set -g status-interval 2
set -g set-clipboard on
setw -g monitor-activity on
set -g visual-activity off

set -g status-style "bg=colour234,fg=colour250"
set -g status-left "#[fg=colour45,bold] #H "
set -g status-right "#[fg=colour244]%Y-%m-%d #[fg=colour221]%H:%M "
set -g window-status-format "#[fg=colour244] #I:#W "
set -g window-status-current-format "#[fg=colour234,bg=colour45,bold] #I:#W "
```

Before setting `tmux-256color`, verify the terminfo entry exists:

```bash
infocmp tmux-256color >/dev/null
```

If it does not exist, stop and resolve terminfo using the installed tmux
distribution. Do not silently claim true-colour support while falling back to a
different terminal type.

Every interactive SSH attach must allocate a pseudo-terminal with `ssh -t`.
Acceptance includes a visible 24-bit colour test inside tmux:

```bash
printf '\033[38;2;255;100;0mTRUECOLOR\033[0m\n'
```

### FR10: Session lifecycle

One tmux session owns one agent. Session names must use only letters, numbers,
dots, underscores, and hyphens.

Recommended naming:

```text
<project>-codex
<project>-claude
<project>-<task>-codex
```

Starting an existing name must attach to it rather than create a duplicate.
Detaching or closing SSH must not stop the agent. Reboot persistence is out of
scope and must not be implied.

### FR11: Repository isolation

Repositories live on the worker. Git, not automatic file synchronization, is
the transfer and collaboration mechanism.

Rules:

- clone repositories independently on the worker;
- do not share one checkout between concurrent writing agents;
- create a dedicated Git worktree or separate clone per concurrent task;
- inspect repository-local `AGENTS.md` before starting an agent;
- preserve dirty work and never reset or delete it as part of machine setup;
- do not commit, push, deploy, migrate, or release unless the task separately
  authorizes it.

Cursor Remote SSH may edit the worker checkout directly. It does not change the
isolation requirement.

### FR12: Status command

Expose status through the dependency-free controller command `side-chick`. It
must support:

```bash
side-chick status
side-chick watch
side-chick watch 5
```

The snapshot must show:

- worker name and time;
- uptime and load;
- CPU and physical memory summary;
- power source and battery state;
- data-volume disk usage;
- Claude and Codex processes with PID, elapsed time, CPU, and memory;
- tmux sessions.

Watch mode should default to a two-second refresh and stop with `Ctrl-C`. Use
ANSI colours only when output is a terminal, produce plain text when piped, and
honour `NO_COLOR`. Never print environment values or process environments.

### FR13: Generic controller helper

After the direct SSH path passes, an optional `side-chick` helper may wrap the
common commands. It must:

- accept an arbitrary remote directory;
- accept only `codex` or `claude` as the agent selector;
- validate and safely quote every user-supplied value;
- refuse a missing remote directory;
- create or attach one tmux session;
- wrap a new agent in `caffeinate -i -m`;
- use `ssh -t` for interactive commands;
- expose direct recovery commands in `side-chick --help`;
- contain no project catalog, tokens, IP addresses, or model configuration.

Do not add this helper until the direct commands have passed. The helper is a
convenience layer, not a dependency.

### FR14: Cursor Remote SSH

Cursor is optional on the controller. If used:

- verify the Remote SSH extension is installed;
- verify the worker alias appears in the remote host picker;
- open a disposable or clean worker checkout;
- confirm the integrated terminal reports the worker hostname;
- confirm colours and Unicode render correctly;
- confirm closing Cursor does not stop a tmux-owned agent.

Do not install remote-container extensions or a container runtime for this PRD.

## Non-functional requirements

### Cost

- Infrastructure software and LAN transport add no recurring cost.
- Existing Claude and Codex eligibility or API billing is outside the
  infrastructure budget.
- Do not introduce a paid networking service to claim completion.

### Performance

- A reused SSH connection should open in approximately two seconds or less on
  a healthy LAN.
- Status watch refreshes every two seconds by default.
- Interactive output should feel immediate; sustained visible lag is a failed
  network acceptance test.

### Reliability

- Closing the controller terminal must not stop a tmux-owned process.
- A short controller network interruption must not corrupt the worker
  checkout.
- The system must expose direct SSH and tmux recovery commands when helpers
  fail.
- Worker reboot or lid-close suspension may stop sessions and is explicitly not
  hidden.

### Security

- LAN only; no public SSH exposure.
- Remote Login restricted to the designated user.
- Dedicated SSH key preferred; private key stays on the controller.
- Agent credentials stay on the worker and are never copied by setup tooling.
- No secrets in dotfiles, helper scripts, logs, PRDs, or setup receipts.
- No full disk access, firewall changes, router changes, or persistent
  LaunchAgents without explicit approval.

## Implementation sequence

### Phase 0: Inspect

1. Read applicable machine and repository instructions.
2. Inventory both Macs without changing them.
3. Fill the operator decision table.
4. Report existing conflicts and the exact proposed diff.

Gate: the operator confirms the worker identity, username, network scope, and
whether dedicated SSH-key creation is allowed.

### Phase 1: Establish native access

1. Set the worker computer name and local hostname.
2. Enable Remote Login for only the designated user.
3. Resolve and validate the worker host fingerprint.
4. Complete one password-authenticated connection.
5. Add the controller SSH alias and validate it with `ssh -G`.

Gate: `ssh <worker-alias>` reaches the correct worker on the LAN.

### Phase 2: Remove connection friction

1. Create and authorize a dedicated key if approved.
2. Verify passwordless login before closing the original session.
3. Enable SSH connection reuse and keepalives.
4. Verify repeated connections and Cursor discovery.

Gate: repeated SSH commands connect without a password and without host-key
warnings.

### Phase 3: Prepare the worker

1. Install only missing minimum packages.
2. Merge the tmux configuration.
3. Verify tmux terminfo and true colour.
4. Install Claude Code and Codex from current official sources.
5. Authenticate each agent using its remote-safe browser flow.

Gate: version, doctor, and authentication status checks pass without exposing
credentials.

### Phase 4: Prove persistence

1. Create a disposable tmux session.
2. Run a visible counter or other harmless long-running command.
3. Detach and close SSH.
4. Reconnect and confirm the process continued.
5. Run one harmless agent interaction in a disposable Git repository.
6. Confirm `caffeinate` produces the expected power assertion while the agent
   is active.

Gate: both Claude Code and Codex can start inside tmux, detach, and reattach.

### Phase 5: Add convenience commands

1. Install the status command on the controller.
2. Validate one-shot, watch, piped, and no-agent states.
3. Add the `side-chick` helper only after direct recovery commands pass.
4. Verify Cursor Remote SSH if requested.

Gate: the operator can run, observe, detach, and reattach without remembering
implementation details.

## Acceptance test matrix

| Test | Expected evidence |
| --- | --- |
| Identity | Controller resolves the intended `.local` hostname |
| SSH configuration | `ssh -G` shows expected host, user, keepalive, and control path |
| Host authenticity | Operator-confirmed Ed25519 host fingerprint |
| Authentication | New SSH connection requires no password when key auth was approved |
| Isolation | Remote Login is limited to the designated user |
| Persistence | Counter or agent continues after controller terminal closes |
| tmux | Session lists, attaches, detaches, and retains large scrollback |
| Colour | 24-bit test renders correctly inside tmux and Cursor terminal |
| Codex | Version and `codex login status` succeed |
| Claude | Version, `claude doctor`, and `claude auth status` succeed |
| Power | `pmset -g assertions` shows the agent's caffeinate assertion |
| Status snapshot | CPU, memory, power, disk, agents, and tmux are present |
| Status watch | Refreshes until `Ctrl-C` without accumulating processes |
| Clean output | Piped status contains no ANSI escapes or secrets |
| Repository safety | Concurrent agents use distinct worktrees/checkouts |
| Recovery | Direct SSH/tmux commands work without helper scripts |
| Exposure | No router port forward or public endpoint exists |

## Definition of done

- [ ] Both machines have recorded non-secret identity values.
- [ ] The worker is reachable through one stable controller alias.
- [ ] Remote Login is restricted to the intended user.
- [ ] Host-key identity was validated.
- [ ] Passwordless SSH works if key creation was approved.
- [ ] SSH keepalive and connection reuse are effective.
- [ ] tmux persistence, scrollback, mouse, clipboard, and true colour work.
- [ ] Claude Code is installed, authenticated, and verified.
- [ ] Codex is installed, authenticated, and verified.
- [ ] Agent-scoped caffeinate behaviour is verified.
- [ ] Status snapshot and watch commands work.
- [ ] Cursor Remote SSH works if requested.
- [ ] One agent survives controller disconnect and reattachment.
- [ ] Concurrent-agent checkout isolation is demonstrated or documented.
- [ ] Direct recovery commands are included in the handoff.
- [ ] No secrets or private host state were written into a repository.

## Stop conditions

The setup agent must stop and ask for direction if:

- the Macs are not on the same trusted LAN;
- the worker username or target machine is ambiguous;
- enabling Remote Login would expose the worker beyond the intended network;
- a host fingerprint changes unexpectedly;
- the requested setup requires reading or copying private credentials;
- existing SSH, tmux, sleep, or management configuration conflicts materially;
- the worker must operate reliably with its lid closed but has no supported
  clamshell arrangement;
- the operator requires off-LAN access while preserving both zero cost and
  minimal complexity;
- installing or authenticating an agent would create new usage charges not
  understood by the operator.

## Rollback

Rollback must remove only changes made by this setup:

1. Detach from or gracefully exit agent sessions. Do not kill unrelated tmux
   sessions.
2. Remove the setup-specific controller SSH host block and its control socket.
3. Remove only the setup-specific public key line from worker
   `authorized_keys`; never delete the entire file.
4. Restore only the tmux lines added by this setup.
5. Remove helper commands created by this setup.
6. Uninstall Claude Code or Codex only if the operator requests it, following
   the current official uninstall guidance.
7. Disable Remote Login if the worker no longer needs SSH access.
8. Remove a persistent caffeinate LaunchAgent only if one was separately
   approved and installed.

Report what was removed and which changes, such as agent credentials or Git
checkouts, were intentionally preserved.

## Agent handoff prompt

Give the following prompt to an agent running on either Mac:

```text
Implement the Side Machine PRD:
docs/PRD.md

First determine whether this is the controller or worker Mac. Read the nearest
AGENTS.md and perform Phase 0 read-only inventory. Do not inspect secrets,
environment files, private SSH keys, or agent credential stores. Report the
non-secret role values and smallest proposed changes before editing.

Keep the solution LAN-only, zero-incremental-cost, and limited to native SSH,
tmux, caffeinate, Claude Code, Codex CLI, Git, and optional Cursor Remote SSH.
Do not install Coder, Colima, Docker, Kubernetes, Tailscale, Headscale,
monitoring services, or a custom orchestrator.

Use current official vendor documentation for agent installation and
authentication. Ask before creating or authorizing an SSH key, enabling a
persistent LaunchAgent, changing firewall/router settings, or adding any paid
service. Preserve existing SSH/tmux configuration and dirty repositories.

Work through the phase gates in order. Stop on identity, host-key, networking,
credential, power/lid, or cost ambiguity. Finish with the acceptance matrix,
exact receipts, recovery commands, and remaining limitations. Do not commit,
push, deploy, migrate, or release unless separately instructed.
```

## Setup receipt template

Agents should return this structure without sensitive values:

```text
Role: controller | worker
Computer name: <non-secret name>
Local hostname: <non-secret hostname>
macOS / architecture: <version> / <architecture>
Worker alias: <alias or n/a>
Network scope: same LAN

Installed or changed:
- <small explicit list>

Validation:
- SSH identity: PASS | FAIL | NOT RUN
- Passwordless login: PASS | DECLINED | NOT RUN
- tmux persistence: PASS | FAIL | NOT RUN
- true colour: PASS | FAIL | NOT RUN
- Codex install/auth: PASS | FAIL | NOT RUN
- Claude install/auth: PASS | FAIL | NOT RUN
- caffeinate assertion: PASS | FAIL | NOT RUN
- status command: PASS | FAIL | NOT RUN
- Cursor Remote SSH: PASS | NOT REQUESTED | NOT RUN

Recovery commands:
- <direct SSH command>
- <tmux list command>
- <tmux attach command>

Remaining limitations:
- LAN only
- worker reboot/lid-close behaviour
- <anything else observed>
```

## Official references

These links were verified on the review date. Setup agents must still check
them for current instructions before execution.

- [Apple: allow a remote computer to access your Mac](https://support.apple.com/guide/mac-help/allow-a-remote-computer-to-access-your-mac-mchlp1066/mac)
- [OpenAI: Codex CLI](https://developers.openai.com/codex/cli)
- [OpenAI: Codex authentication](https://developers.openai.com/codex/auth)
- [Anthropic: Claude Code installation](https://code.claude.com/docs/en/installation)
- [Anthropic: Claude Code authentication](https://code.claude.com/docs/en/authentication)
- [Anthropic: Claude Code CLI reference](https://code.claude.com/docs/en/cli-usage)
- [tmux source and documentation](https://github.com/tmux/tmux)
