# Day 1 — Linux Filesystem & Navigation

## Environment Check
- OS: Ubuntu 24.04 LTS (via WSL2)
- Confirmed with: uname -a, lsb_release -a

## FHS — What I Found

| Folder | What lives there (in my own words) |
|--------|-------------------------------------|
| /etc   | Config files of everything — saw things like nsswitch.conf, resolv.conf |
| /var   | Logs, cache, spool — varies ,grows over time |
| /home  | My user folder |
| /usr   | Most installed programs actually live here |
| /bin   | Core commands like ls, cp, mv |
| /opt   | Third-party/optional software |

## Navigation Commands I Practiced
- pwd — shows current location
- cd / cd ~ / cd -  — move around, jump home, jump back
- ls -la — list all files including hidden ones
- find / -name "*.conf" 2>/dev/null | head -10 — search whole system for config files, hide permission errors
- tree -L 2 /etc — visual folder tree, 2 levels deep

## Blind Navigation — What Surprised Me
1. (e.g.) /root is off-limits without sudo — good reminder about permissions
2. (e.g.) /usr/bin has way more binaries than I expected — ran `ls | wc -l`
3. (e.g.) Didn't realize /proc isn't real files — it's a live view into running processes
4. (e.g.) [add your own]

## Notes to self
- I was logged in as root at one point — need to check why / switch back to regular user
- Installed `tree` today
 
