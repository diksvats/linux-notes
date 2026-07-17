# Day 5: Package Management (apt) & systemd/Services

## Package Management with apt

### `apt list --installed | wc -l`
Lists all installed packages and pipes the output to `wc -l` to count lines.
Result on this system: **524 packages installed**.

### `apt-cache policy <package>`
Shows the installed version vs. the candidate version (what apt would install on upgrade), plus a version table listing every version known across configured repos, with priority numbers.

Example — `apt-cache policy curl`:
- `Installed` and `Candidate` matched → curl was already up to date.
- Version table showed two versions: the patched one (from `noble-updates` and `noble-security`) and the original release version (from `noble`), demonstrating how Ubuntu layers patches on top of the base release.

### `sudo apt update`
Refreshes the local package **index/catalog** from all configured repos. Does **not** install or upgrade anything — it only updates apt's knowledge of what's available.

Repo/component structure observed:
- Repos: `noble` (base release), `noble-updates`, `noble-security`, `noble-backports`
- Components: `main` (officially supported, open-source), `universe` (community, open-source), `restricted` (supported, not fully open-source), `multiverse` (neither)

Output ended with: `39 packages can be upgraded.`

### `apt list --upgradable`
Lists packages with a newer candidate version available. Reviewed all 39 — mostly routine patch-level bumps (e.g. `9.4-3ubuntu6.1` → `9.4-3ubuntu6.2`), covering:
- Core system tools: `binutils*`, `coreutils`, `apparmor`
- Networking: `iproute2`, `netplan*`
- Familiar CLI tools: `vim*`, `tar`, `wget`, `xxd`
- Crash reporting: `apport*`
- Cloud provisioning: `cloud-init` (configures VMs on first boot — relevant for AWS/GCP later)
- Package management itself: `snapd`, `software-properties-common`

### `sudo apt upgrade`
Applied all 39 pending upgrades. Confirmed clean by re-running `apt list --upgradable` → returned empty (nothing pending).

**Key distinction:** `apt update` = refresh the catalog. `apt upgrade` = actually install newer versions of already-installed packages. Easy to conflate — they are two separate steps.

---

## systemd & Services

### Installing and inspecting a service
```
sudo apt install openssh-server
systemctl status ssh
```

Initial state before starting:
```
Loaded: loaded (...; disabled; preset: enabled)
Active: inactive (dead)
```

### `sudo systemctl start ssh`
Starts the service immediately. Status changed to:
```
Active: active (running) since <timestamp>
Main PID: 3961 (sshd)
```
Also showed:
- `ExecStartPre=/usr/sbin/sshd -t` — systemd validates the config syntax *before* starting the daemon.
- Resource tracking built in: Tasks, Memory, CPU, CGroup — systemd tracks this per-service natively.
- Log tail showing sshd listening on port 22 (both IPv4 `0.0.0.0` and IPv6 `::`).

### `sudo systemctl enable ssh`
Enables the service to auto-start on boot. Under the hood this is **just symlinks**, not a separate config database:
- `/etc/systemd/system/sshd.service` → alias symlink
- `/etc/systemd/system/multi-user.target.wants/ssh.service` → the real one; `multi-user.target` = normal running system with networking, no GUI (like old runlevel 3). Anything symlinked into its `.wants/` directory starts when that target is reached.

**stop vs. disable — two independent axes:**
| | Affects |
|---|---|
| `systemctl stop` | Current running state only (right now) |
| `systemctl disable` | Boot behavior only (next boot) |

A service can be running+enabled, running+disabled, stopped+enabled, or stopped+disabled — these are independent settings.

### `sudo journalctl -u ssh`
Filters the systemd journal to just this unit's log history. Journal entries accumulate over time (not cleared between start/stop cycles).

Observed a stop/start cycle:
```
sshd[3961]: Received signal 15; terminating.
systemd[1]: Stopping ssh.service...
systemd[1]: ssh.service: Deactivated successfully.
systemd[1]: Stopped ssh.service...
```
**Signal 15 = SIGTERM** — a graceful shutdown request (same default signal plain `kill` sends, from Day 4 notes), as opposed to `SIGKILL` (signal 9, forced/no cleanup).

New start after stop showed a **new PID** (4122 vs 3961), confirming a genuinely new process rather than the old one resuming.

---

## Key takeaways
- `apt update` (refresh catalog) vs `apt upgrade` (apply newer versions) — separate steps, don't conflate them.
- Ubuntu repo structure: repo (`noble`, `noble-updates`, `noble-security`, `noble-backports`) × component (`main`, `universe`, `restricted`, `multiverse`).
- systemd's "enable" = symlink into `multi-user.target.wants/`; no hidden state beyond the filesystem.
- `stop`/`start` = now; `enable`/`disable` = on boot — independent controls.
- `journalctl -u <service>` is the tool for full per-service log history, beyond what `systemctl status` shows in its tail.
