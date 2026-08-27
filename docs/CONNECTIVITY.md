# Device connectivity

Side Machine uses one private Tailscale tailnet as a three-device mesh. No
device needs to share Wi-Fi, expose SSH to the public internet, or configure
router port forwarding.

## Topology

```text
                         private Tailscale tailnet

  iPhone: iphone171
  Tailscale + SSH client
          │
          ├──────────────────────► M5 Mac (primary/controller)
          │                         sarthaks-macbook-pronew
          │
          └──────────────────────► M1 Mac (side machine/worker)
                                    fleet-mac

  M5 Mac ───── side-chick / SSH ─► M1 Mac
```

Tailscale provides the private network path. SSH provides the terminal. The
Tailscale iOS app does not itself provide a shell, so phone-to-Mac terminal
access also needs an iOS SSH client.

## Current device inventory

| Role | Tailscale name | Current tailnet IP | State last verified |
| --- | --- | --- | --- |
| M5 primary Mac | `sarthaks-macbook-pronew` | `100.125.168.76` | Online and reachable from M1 |
| M1 side machine | `fleet-mac` | `100.109.46.74` | Online; Tailscale SSH, Herdr, and Caffeinate running |
| iPhone | `iphone171` | `100.114.21.63` | Registered but offline; open Tailscale on the phone |

Prefer Tailscale names over IP addresses. An address can change if a device is
removed from the tailnet and enrolled again.

## Connection matrix

| From | To | Network path | Terminal readiness |
| --- | --- | --- | --- |
| M5 Mac | M1 Mac | Verified through Tailscale | Ready: use `side-chick` |
| M1 Mac | M5 Mac | Verified through Tailscale relay | Network works; inbound SSH on M5 is not yet verified |
| iPhone | M1 Mac | Available when `iphone171` is online | M1 is ready; connect an SSH client as `assistant@fleet-mac` |
| iPhone | M5 Mac | Available when `iphone171` is online | Enable Remote Login on M5 and use its macOS account name |

The M5-to-M1 test used a Tailscale DERP relay. That is a valid encrypted path:
a direct peer-to-peer route is preferable for latency but is not required for
connectivity.

## M5 Mac to M1 Mac

The M5 is the controller and the M1 is the worker:

```bash
side-chick alive
side-chick status
side-chick speed
side-chick herdr
side-chick
```

`side-chick` defaults to `assistant@fleet-mac`. Use `side-chick lan` only for
same-LAN recovery.

## iPhone to M1 Mac

1. Open Tailscale on the iPhone and connect it to the same tailnet.
2. Open an iOS SSH client.
3. Create a host with hostname `fleet-mac`, username `assistant`, and port 22.
4. Connect and verify the host key on first use.

Inside the M1 shell, the same worker commands are available:

```bash
side-machine-status
side-machine-speed
cd ~/code && herdr
```

## iPhone to M5 Mac

1. On the M5, open **System Settings → General → Sharing**.
2. Enable **Remote Login** and restrict access to the intended macOS user.
3. Keep Tailscale connected on the M5.
4. On the iPhone SSH client, create a host named
   `sarthaks-macbook-pronew`, using the permitted M5 macOS username and port
   22.

Do not add public port forwarding. The SSH service should be reached only over
Tailscale or a trusted local network.

## Availability conditions

Internet access alone is not sufficient. A destination Mac must also be awake,
connected to Tailscale, authorized in the tailnet, and running an SSH service.
The M1 has persistent Caffeinate and Herdr services, but closing a MacBook lid
can still suspend it unless normal macOS clamshell requirements are met.

## Authentication and secrets

- Never place a Mac password in `side-chick`, an SSH profile, or a shell alias.
- Tailscale SSH can use tailnet identity when its access policy permits it.
- Otherwise, use a dedicated SSH key and install only its public half on the
  destination Mac.
- A first-use host-key prompt is normal. A later changed-key warning should be
  investigated rather than bypassed.

## Quick checks

From the M5:

```bash
tailscale status
tailscale ping fleet-mac
side-chick alive
```

From either Mac, `tailscale status` should show all currently connected peers.
On the iPhone, the Tailscale app should show `fleet-mac` and
`sarthaks-macbook-pronew` as reachable before opening the SSH client.
