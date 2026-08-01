# Day 2 — Permissions & Ownership Lab

## 1. Concepts

- rwx notation: -rwxr-xr-- = type, owner perms, group perms, other perms
- Octal notation: r=4, w=2, x=1, summed per group (e.g. rwx=7, r-x=5, r--=4, rw-=6)
- chmod — change file permissions (symbolic or octal)
- chown — change file owner (and optionally group)
- chgrp — change group only

## 2. NTFS ACL Comparison

| Windows NTFS | Linux |
|---|---|
| Per-user, stackable ACLs (multiple named users/groups per file) | One owner + one group per file |
| Managed via GUI (Properties -> Security) or icacls | Managed via chmod/chown/chgrp |
| Inheritable permissions down folder trees | No native inheritance; must be set explicitly per file/folder or via umask |
| Granular rights (Modify, Read & Execute, Special Permissions, etc.) | Simpler 3-tier model: read/write/execute only |

Linux trades granularity for simplicity — fewer permission combinations, but faster to reason about and script.

## 3. Break/Fix Lab — Before/After

Baseline (all files created with touch):
-rw-r--r-- 1 diksha diksha 0 file1.txt   (repeated for file1-file10)

After chmod changes:

| File | Command | Result | Meaning |
|---|---|---|---|
| file1.txt | chmod 700 | -rwx------ | Owner full access, no one else |
| file2.txt | chmod u+x | -rwxr--r-- | Added execute for owner only |
| file3.txt | chmod go-rwx | -rw------- | Stripped all perms for group & others |
| file4.txt | chmod 644 | -rw-r--r-- | Standard file: owner rw, everyone else read |
| file5.txt | chmod 666 | -rw-rw-rw- | Everyone read+write — risky, no execute |
| file6.txt | chmod 777 | -rwxrwxrwx | Full access for everyone — bad practice, demo only |
| file7.txt | chmod o-r | -rw-r----- | Removed read for others only |
| file8.txt | chmod g+w | -rw-rw-r-- | Added write for group |
| file9.txt | chmod 000 | ---------- | No permissions at all, even for owner |
| file10.txt | chmod 755 | -rwxr-xr-x | Owner full, group/others read+execute — typical script perms |

Key observation — file9.txt:
Running cat file9.txt after chmod 000 returned Permission denied, even as the file's owner. Confirms permission bits apply regardless of ownership.

Key observation — file10.txt:
After setting execute permission and adding a script line (echo "hello from script"), running ./file10.txt successfully executed the file. Confirmed execute bit is required to run a file as a program, and ./ is needed because Linux doesn't search the current directory by default for security reasons.

## 4. Shared-Group Access Scenario

Commands used:
sudo groupadd devteam
sudo usermod -aG devteam diksha
newgrp devteam
mkdir ~/shared-project
sudo chgrp devteam shared-project
chmod 770 shared-project

Result:
drwxrwx--- 2 diksha devteam 4096 shared-project

Meaning:
- Owner (diksha): full access
- Group (devteam): full access
- Others: no access

Real-world application: on a multi-user server, any teammate added to the devteam group automatically gets full read/write/execute access to this folder without being its owner — e.g. a shared deployment scripts folder or team config directory. Compared to NTFS, this is roughly equivalent to assigning a security group "Modify" rights on a shared drive, except Linux only supports one group per resource rather than stacking multiple named groups.

## 5. Takeaways

- Octal notation is faster to use in practice than symbolic once memorized
- chmod 000 blocks even the owner — ownership does not mean automatic access
- Group-based sharing (chgrp + correct octal) is the standard pattern for multi-user access control on Linux servers
