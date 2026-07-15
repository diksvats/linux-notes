# Day 4: Process Management & System Monitoring

## Commands

### Viewing processes
- `ps aux` — snapshot of all running processes: user, PID, %CPU, %MEM, command
- `ps -ef` — same idea, but includes PPID (parent process ID) — useful for seeing what started a process
- `pstree` — visual tree of parent/child process relationships

### Live monitoring
- `top` — real-time process viewer. Key interactive commands: `M` sort by memory, `P` sort by CPU, `k` kill a process, `q` quit
- `htop` — nicer UI version of `top` (install via `sudo apt install htop`)

### Process control
- `kill <PID>` — sends SIGTERM ("please stop gracefully")
- `kill -9 <PID>` — sends SIGKILL (force kill, last resort)
- `killall <name>` — kill by process name instead of PID

### Jobs & backgrounding
- `command &` — start a process already in the background
- `jobs` — list background jobs in the current shell session
- `Ctrl+Z` — suspend (pause) a running foreground process
- `bg` — resume a suspended process, but in the background
- `fg` — bring a background job back to the foreground
- `nohup command &` — keep a process running even after logging out

### Resource snapshots
- `free -h` — memory usage (human-readable)
- `df -h` — disk usage per filesystem
- `uptime` — system load average at a glance

## Signals mini-table

| Signal | Number | Behavior | When to use |
|---|---|---|---|
| SIGTERM | 15 | Asks process to exit gracefully (default for `kill`) | Normal shutdown of a process |
| SIGKILL | 9 | Forces immediate termination, no cleanup | Process is unresponsive to SIGTERM |
| SIGHUP | 1 | Originally "terminal hung up"; often used to tell a daemon to reload its config | Reloading a running service without restarting it |

## Exercise notes

### 1. Finding my own shell process (PID vs PPID)
```
$ ps aux | grep bash
diksha  339  0.0  0.1  6212  5472 pts/0  Ss  16:34  0:00 -bash
$ echo $$
339
$ ps -ef | grep 339
diksha  339  338  0  16:34 pts/0  00:00:00 -bash
diksha  625  339  0  17:11 pts/0  00:00:00 ps -ef
diksha  626  339  0  17:11 pts/0  00:00:00 grep --color=auto 339
```
- My shell's PID is 339, its PPID is 338 (the process that launched it).
- Every command I run (like `ps -ef` and `grep` above) spawns as a *child* of my shell (PPID 339), runs briefly, then exits. This is the core insight: the shell is a parent process constantly spawning short-lived children.

### 2. Backgrounding and killing a process
```
$ sleep 300 &
[1] 686
$ ps aux | grep sleep
diksha  686  0.0  0.0  3132  1972 pts/0  S  17:34  0:00 sleep 300
$ kill 686
$ jobs
[1]+  Terminated              sleep 300
```
- `sleep 300 &` started immediately in the background, freeing up my terminal.
- Found the PID with `ps aux | grep sleep`, matched the job number from `jobs`.
- `kill 686` sent SIGTERM, and the process terminated cleanly.

### 3. Suspending with Ctrl+Z, then resuming in background
```
$ sleep 300
^Z
[1]+  Stopped                 sleep 300
$ bg
$ jobs
[1]+  Running                 sleep 300 &
$ ps aux | grep sleep
$ kill <PID>
$ jobs
```
- Starting `sleep 300` without `&` ties up the terminal in the foreground.
- `Ctrl+Z` suspends (pauses) it rather than killing it — status shows `Stopped`.
- `bg` resumes the same process, but now running in the background — status flips to `Running`.
- Found and killed it the same way as before.

### 4. System resource snapshot
```
$ free -h
Mem:  total 3.7Gi  used 380Mi  free 3.2Gi  available 3.3Gi
Swap: total 1.0Gi  used 0B    free 1.0Gi

$ df -h
/dev/sdd  1007G  1.6G  955G   1%  /
C:\        164G   99G   66G  60%  /mnt/c
D:\        313G  105M  313G   1%  /mnt/d

$ uptime
17:51:47 up 1:17, 1 user, load average: 0.07, 0.02, 0.00
```
- Memory: only 380 MB of 3.7 GB used, no swap in use — WSL2 environment is idle/healthy.
- Disk: Linux root filesystem has 955 GB free; Windows C: drive is at 60% used (worth monitoring long-term, not urgent).
- Load average near zero across 1/5/15-min windows — confirms an idle system with nothing CPU-hungry running.

## SRE relevance

- **Diagnosing a runaway process**: `top`/`ps aux` sorted by CPU or memory is the first move when a server "feels slow" — find what's actually consuming resources before doing anything else.
- **Safely restarting a hung service**: `kill` (SIGTERM) first to allow graceful shutdown; only escalate to `kill -9` (SIGKILL) if the process refuses to die, since SIGKILL skips cleanup and can leave resources in a bad state.
- **Long-running jobs during on-call/maintenance work**: `nohup command &` or `Ctrl+Z` + `bg` lets me start something long-running (a backup, a migration script) and keep working without losing it when I disconnect.
- **Baseline health checks**: `free -h`, `df -h`, and `uptime` are the fastest three commands to run when triaging "is this server okay?" — memory pressure, disk space, and load average are the first three things to rule in or out.
