# Versioning and releases

Side Machine follows semantic versioning for its operator-facing command and
transport behavior. Releases are represented by annotated Git tags and a
matching repository-root `VERSION` file.

## Release line

| Release | Transport | Command surface | Purpose |
| --- | --- | --- | --- |
| `v0.0.0` | Same-LAN design only | PRD only | Original specification baseline |
| `v1.0.0` | Bonjour / `.local` SSH | `side-chick`, health, speed, Herdr, key setup | First working implementation |
| `v2.0.0` | Tailscale by default, LAN fallback | Same stable commands | Work across different networks |

## Compatibility policy

- `side-chick` with no arguments always opens a shell.
- Existing v1 command names remain valid in v2.
- New transport behavior must not expose SSH publicly or add router port
  forwarding.
- Credentials, private keys, auth keys, tailnet policy, and device secrets are
  never committed.
- tmux remains a recovery fallback even though Herdr is the primary workspace
  manager from v1 onward.

## Release procedure

1. Update `VERSION`, `CHANGELOG.md`, and the matching file under
   `docs/versions/`.
2. Run shell syntax checks and worker acceptance checks.
3. Commit the release as `release: vX.Y.Z`.
4. Create an annotated `vX.Y.Z` tag on that commit.
5. Push the commit and tag only after explicit owner approval.

## Rollback

Tags are immutable recovery points. Inspect or restore a release without
rewriting the current checkout:

```bash
git show v1.0.0
git worktree add ../side-machine-v1 v1.0.0
```
