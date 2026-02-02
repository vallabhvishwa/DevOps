# The Complete DevOps Engineer's Reference Guide
## Part 2: Linux Advanced - Users, Services, Storage & Performance

---

# Chapter 3: User and Permission Management

## 3.1 Understanding Users and Groups

Linux is a multi-user operating system. Every file and process is owned by a user and associated with a group. This ownership model is fundamental to Linux security.

### User Types

```
User Types in Linux:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. Root User (Superuser)                                   │
│     ├── UID: 0                                              │
│     ├── Has complete system access                          │
│     ├── Can read/write/execute any file                     │
│     ├── Can kill any process                                │
│     ├── Can change any configuration                        │
│     └── Should be used sparingly (use sudo instead)         │
│                                                             │
│  2. System Users (Service Accounts)                         │
│     ├── UID: 1-999 (or 1-499 on older systems)             │
│     ├── Created for running services/daemons                │
│     ├── Usually cannot log in (shell set to /sbin/nologin) │
│     ├── Examples: www-data, mysql, nginx, sshd              │
│     └── Each service runs as its own user for isolation     │
│                                                             │
│  3. Regular Users                                           │
│     ├── UID: 1000+ (or 500+ on older systems)              │
│     ├── Human users who log into the system                 │
│     ├── Each has a home directory (/home/username)          │
│     └── Limited permissions, need sudo for admin tasks      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### User Management Commands

```bash
# View current user
whoami                        # Print effective username
id                            # Print user ID, group ID, and groups
id username                   # Info for specific user

# Example output of 'id':
# uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),999(docker)
# This shows: UID, primary GID, and all supplementary groups

# List all users
cat /etc/passwd               # View user database
getent passwd                 # Query user database (includes LDAP, etc.)
getent passwd john            # Query specific user

# Understanding /etc/passwd format:
# john:x:1001:1001:John Doe:/home/john:/bin/bash
# [1] :[2]:[3] :[4] :  [5]  :   [6]    :  [7]
#
# [1] Username
# [2] Password placeholder (actual password in /etc/shadow)
# [3] User ID (UID)
# [4] Primary Group ID (GID)
# [5] GECOS field (comment, usually full name)
# [6] Home directory
# [7] Login shell

# Create user
useradd username              # Create user (minimal)
useradd -m username           # Create user with home directory
useradd -m -s /bin/bash username           # With specific shell
useradd -m -s /bin/bash -G sudo,docker username  # With groups
useradd -m -s /bin/bash -c "John Doe" john       # With comment

# Full example with all common options:
useradd \
  -m \                        # Create home directory
  -d /home/john \             # Specify home directory
  -s /bin/bash \              # Login shell
  -c "John Doe" \             # Comment (full name)
  -G sudo,docker \            # Supplementary groups
  -e 2024-12-31 \             # Account expiration date
  -u 1500 \                   # Specific UID
  john

# Create system user (for services)
useradd -r -s /sbin/nologin servicename
# -r : Create system user (UID < 1000)
# -s /sbin/nologin : Cannot log in interactively

# Modify user
usermod -aG docker john       # Add to group (-a = append, very important!)
usermod -G wheel,users john   # Set groups (REPLACES existing groups!)
usermod -s /bin/zsh john      # Change shell
usermod -d /home/newdir john  # Change home directory
usermod -l newname oldname    # Rename user
usermod -L john               # Lock account (disable login)
usermod -U john               # Unlock account
usermod -e 2024-12-31 john    # Set expiration date
usermod -e "" john            # Remove expiration

# CAUTION: usermod -G without -a removes user from all other groups!
# ALWAYS use -aG when adding to a group

# Delete user
userdel username              # Delete user (keep home directory)
userdel -r username           # Delete user AND home directory
userdel -f username           # Force delete (even if logged in)

# Set/change password
passwd                        # Change own password
passwd username               # Change another user's password (as root)
passwd -l username            # Lock account
passwd -u username            # Unlock account
passwd -d username            # Delete password (passwordless login - dangerous!)
passwd -e username            # Expire password (force change on next login)
passwd -S username            # Show password status

# Password aging
chage -l username             # List password aging info
chage -M 90 username          # Password expires every 90 days
chage -m 7 username           # Minimum 7 days between changes
chage -W 14 username          # Warn 14 days before expiration
chage -E 2024-12-31 username  # Account expires on date
chage -d 0 username           # Force password change on next login

# Switch user
su - username                 # Switch to user (login shell)
su username                   # Switch to user (non-login shell)
su -                          # Switch to root
sudo -i                       # Interactive root shell
sudo -u username command      # Run command as another user
```

### Group Management

Groups provide a way to assign permissions to multiple users at once.

```bash
# View groups
groups                        # Groups for current user
groups username               # Groups for specific user
cat /etc/group                # View group database
getent group                  # Query group database

# Understanding /etc/group format:
# developers:x:1002:john,jane,bob
# [1]       :[2]:[3]:[4]
#
# [1] Group name
# [2] Group password (rarely used, x = in /etc/gshadow)
# [3] Group ID (GID)
# [4] Group members (comma-separated)

# Create group
groupadd groupname            # Create group
groupadd -g 2000 groupname    # Create with specific GID

# Modify group
groupmod -n newname oldname   # Rename group
groupmod -g 3000 groupname    # Change GID

# Delete group
groupdel groupname            # Delete group

# Manage group membership
gpasswd -a username groupname # Add user to group
gpasswd -d username groupname # Remove user from group
gpasswd -M john,jane,bob groupname  # Set member list

# After adding user to group, user must log out and back in
# Or use: newgrp groupname    # Activate new group in current session
```

### Primary vs Supplementary Groups

```
Understanding Groups:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Primary Group                                              │
│  ├── Each user has exactly ONE primary group                │
│  ├── Stored in 4th field of /etc/passwd                     │
│  ├── New files created by user belong to this group         │
│  └── Usually same name as username (user private group)     │
│                                                             │
│  Supplementary Groups                                       │
│  ├── User can belong to MANY supplementary groups           │
│  ├── Listed in /etc/group file                              │
│  └── Used to grant additional permissions                   │
│                                                             │
│  Example:                                                   │
│  User 'john' with:                                          │
│  - Primary group: john (GID 1001)                           │
│  - Supplementary groups: sudo, docker, developers           │
│                                                             │
│  When john creates a file:                                  │
│  -rw-r--r-- 1 john john 0 Jan 15 10:00 newfile             │
│                   ^^^^                                      │
│                   Primary group                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 3.2 File Permissions

Every file and directory has three sets of permissions for three categories of users.

### Understanding Permission Notation

```
Permission Display:
-rwxr-xr-x  1  john  developers  4096  Jan 15 10:00  script.sh
│└┬┘└┬┘└┬┘  │   │        │        │         │           │
│ │  │  │   │   │        │        │         │           └── Filename
│ │  │  │   │   │        │        │         └── Modification time
│ │  │  │   │   │        │        └── Size in bytes
│ │  │  │   │   │        └── Group owner
│ │  │  │   │   └── User owner
│ │  │  │   └── Number of hard links
│ │  │  └── Permissions for Others (everyone else)
│ │  └── Permissions for Group
│ └── Permissions for User (owner)
└── File type

File Types (first character):
-  : Regular file
d  : Directory
l  : Symbolic link
b  : Block device (disk)
c  : Character device (terminal, keyboard)
s  : Socket
p  : Named pipe (FIFO)

Permissions (9 characters):
Position 1-3: User (owner) permissions
Position 4-6: Group permissions
Position 7-9: Other (world) permissions

Each set of three:
r (read)    = 4
w (write)   = 2
x (execute) = 1
- (none)    = 0
```

### What Each Permission Means

```
Permissions for FILES:
┌────────────┬────────────────────────────────────────────────┐
│ Permission │ Meaning                                        │
├────────────┼────────────────────────────────────────────────┤
│ r (read)   │ Can view file contents (cat, less, head, etc.) │
│ w (write)  │ Can modify file contents                       │
│ x (execute)│ Can run file as a program                      │
└────────────┴────────────────────────────────────────────────┘

Permissions for DIRECTORIES:
┌────────────┬────────────────────────────────────────────────┐
│ Permission │ Meaning                                        │
├────────────┼────────────────────────────────────────────────┤
│ r (read)   │ Can list directory contents (ls)               │
│ w (write)  │ Can create/delete files in directory           │
│ x (execute)│ Can enter directory (cd) and access contents   │
└────────────┴────────────────────────────────────────────────┘

Common permission combinations for directories:
rwx (7) : Full control - list, create, delete, enter
r-x (5) : Read and enter, but cannot create/delete
--x (1) : Enter only (can access files if you know names)
--- (0) : No access
```

### Numeric (Octal) Permissions

```
Calculating Numeric Permissions:
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  r = 4    w = 2    x = 1                                   │
│                                                            │
│  rwx = 4 + 2 + 1 = 7                                       │
│  rw- = 4 + 2 + 0 = 6                                       │
│  r-x = 4 + 0 + 1 = 5                                       │
│  r-- = 4 + 0 + 0 = 4                                       │
│  -wx = 0 + 2 + 1 = 3                                       │
│  -w- = 0 + 2 + 0 = 2                                       │
│  --x = 0 + 0 + 1 = 1                                       │
│  --- = 0 + 0 + 0 = 0                                       │
│                                                            │
└────────────────────────────────────────────────────────────┘

Common Numeric Permissions:
┌──────┬───────────┬──────────────────────────────────────────┐
│ Mode │ Symbolic  │ Use Case                                 │
├──────┼───────────┼──────────────────────────────────────────┤
│ 755  │ rwxr-xr-x │ Executables, directories (public read)   │
│ 750  │ rwxr-x--- │ Executables, directories (group access)  │
│ 700  │ rwx------ │ Private executables/directories          │
│ 644  │ rw-r--r-- │ Regular files (public read)              │
│ 640  │ rw-r----- │ Config files (group can read)            │
│ 600  │ rw------- │ Private files, SSH keys                  │
│ 400  │ r-------- │ Read-only private files                  │
│ 777  │ rwxrwxrwx │ AVOID! Everyone has full access          │
│ 666  │ rw-rw-rw- │ AVOID! Everyone can read/write           │
└──────┴───────────┴──────────────────────────────────────────┘
```

### Changing Permissions (chmod)

```bash
# chmod - Change file mode (permissions)

# Numeric mode
chmod 755 file                # rwxr-xr-x
chmod 644 file                # rw-r--r--
chmod 600 file                # rw-------
chmod 700 directory           # rwx------

# Symbolic mode
chmod u+x file                # Add execute for user (owner)
chmod g+w file                # Add write for group
chmod o-r file                # Remove read for others
chmod a+x file                # Add execute for all (a = all = ugo)
chmod ug=rw,o=r file          # Set specific permissions
chmod u=rwx,go=rx file        # Owner full, others read+execute

# Symbolic mode syntax:
# WHO:  u (user/owner), g (group), o (others), a (all)
# WHAT: + (add), - (remove), = (set exactly)
# PERM: r (read), w (write), x (execute)
#       s (setuid/setgid), t (sticky bit)
#       X (execute only if directory or already executable)

# Recursive
chmod -R 755 directory        # Apply to directory and all contents
chmod -R u+rwX,go+rX,go-w directory  
# X = execute only for directories (safer than x)

# Reference another file
chmod --reference=ref_file target_file

# Verbose
chmod -v 755 file             # Show what changed
```

### Special Permissions

```
Special Permission Bits:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  SUID (Set User ID) - Numeric: 4000                        │
│  ├── Only for files (executables)                          │
│  ├── When executed, runs with owner's permissions          │
│  ├── Displayed as 's' in user execute position             │
│  ├── Example: -rwsr-xr-x (passwd command)                  │
│  └── Use case: Allow normal users to perform root tasks    │
│                                                             │
│  SGID (Set Group ID) - Numeric: 2000                       │
│  ├── For files: runs with group's permissions              │
│  ├── For directories: new files inherit directory's group  │
│  ├── Displayed as 's' in group execute position            │
│  └── Use case: Shared directories where all files          │
│      should belong to same group                           │
│                                                             │
│  Sticky Bit - Numeric: 1000                                │
│  ├── Only for directories                                  │
│  ├── Only file owner can delete their files                │
│  ├── Displayed as 't' in others execute position           │
│  ├── Example: drwxrwxrwt (/tmp)                            │
│  └── Use case: Shared directories where users shouldn't    │
│      delete each other's files                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Examples:
-rwsr-xr-x  : SUID set, s in user position
-rwxr-sr-x  : SGID set, s in group position
drwxrwxrwt  : Sticky bit set, t in others position

If underlying permission was not x:
-rwSr-xr-x  : SUID set, S (capital) = no underlying x permission
-rwxr-Sr-x  : SGID set, S = no underlying x permission
drwxrwxrwT  : Sticky bit, T = no underlying x permission
```

```bash
# Setting special permissions
chmod u+s file                # Set SUID
chmod g+s directory           # Set SGID
chmod +t directory            # Set sticky bit

chmod 4755 file               # SUID + rwxr-xr-x
chmod 2755 directory          # SGID + rwxr-xr-x
chmod 1777 directory          # Sticky + rwxrwxrwx

# Remove special permissions
chmod u-s file                # Remove SUID
chmod g-s directory           # Remove SGID
chmod -t directory            # Remove sticky bit

# Find files with special permissions
find / -perm -4000 2>/dev/null  # Find SUID files
find / -perm -2000 2>/dev/null  # Find SGID files
find / -perm -1000 2>/dev/null  # Find sticky bit directories
```

### Changing Ownership (chown, chgrp)

```bash
# chown - Change file owner and group

chown user file               # Change owner
chown user:group file         # Change owner and group
chown :group file             # Change group only
chown -R user:group directory # Recursive

# chgrp - Change group ownership
chgrp group file              # Change group
chgrp -R group directory      # Recursive

# Examples
chown john:developers file.txt
chown -R www-data:www-data /var/www/
chown --reference=ref_file target_file  # Copy ownership

# Only root can change owner
# Regular users can only change group (to a group they belong to)
```

### Access Control Lists (ACLs)

ACLs provide more fine-grained permissions than traditional Unix permissions.

```bash
# Check if ACLs are supported
mount | grep acl              # Check mount options
tune2fs -l /dev/sda1 | grep "Default mount options"

# View ACLs
getfacl file                  # Display ACL

# Example output:
# file: myfile
# owner: john
# group: users
# user::rw-
# user:jane:r--
# group::r--
# group:developers:rw-
# mask::rw-
# other::---

# Set ACL
setfacl -m u:jane:rw file            # Give jane read/write
setfacl -m u:bob:r file              # Give bob read only
setfacl -m g:developers:rwx file     # Give developers group full access
setfacl -m o::--- file               # Remove all permissions for others

# ACL syntax: [d:]type:name:permissions
# type: u (user), g (group), o (other), m (mask)
# d: means default (for directories)

# Default ACLs (for new files in directory)
setfacl -d -m u:jane:rw directory    # Default ACL for new files
setfacl -d -m g:developers:rwx directory

# Remove ACL entries
setfacl -x u:jane file               # Remove jane's ACL
setfacl -x g:developers file         # Remove developers group ACL
setfacl -b file                      # Remove all ACLs

# Recursive
setfacl -R -m u:jane:rw directory    # Recursive
setfacl -R -d -m u:jane:rw directory # Recursive default

# Copy ACLs
getfacl file1 | setfacl --set-file=- file2

# Mask
# The mask limits the maximum permissions for named users and groups
# Effective permissions = ACL entry & mask
setfacl -m m::r file                 # Set mask to read-only
# This means even if jane has rw, effective is only r

# ACL on files shows + in ls -l:
# -rw-r--r--+ 1 john users 0 Jan 15 10:00 file
#           ^ indicates ACL present
```

### Default Permissions (umask)

```bash
# umask - Set default permission mask

# umask SUBTRACTS permissions from maximum
# Maximum for files: 666 (no execute by default)
# Maximum for directories: 777

# Common umask values:
# umask 022 : Files 644, directories 755 (default)
# umask 027 : Files 640, directories 750
# umask 077 : Files 600, directories 700

umask                         # Show current umask
umask 022                     # Set umask for current session
umask -S                      # Show in symbolic format

# Calculation:
# umask 022
# Files:       666 - 022 = 644 (rw-r--r--)
# Directories: 777 - 022 = 755 (rwxr-xr-x)
#
# umask 077
# Files:       666 - 077 = 600 (rw-------)
# Directories: 777 - 077 = 700 (rwx------)

# Set permanent umask in:
# ~/.bashrc           (per user)
# /etc/profile        (system-wide)
# /etc/login.defs     (for new users)
```

---

# Chapter 4: System Services and Systemd

## 4.1 Understanding Systemd

Systemd is the modern init system and service manager for Linux. It was adopted by most major distributions around 2015 and replaces older init systems like SysVinit and Upstart.

### What Systemd Does

```
Systemd Responsibilities:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. System Initialization                                   │
│     ├── First process (PID 1)                               │
│     ├── Mounts filesystems                                  │
│     ├── Starts essential services                           │
│     └── Sets up the user environment                        │
│                                                             │
│  2. Service Management                                      │
│     ├── Start/stop/restart services                         │
│     ├── Enable/disable services at boot                     │
│     ├── Monitor service health                              │
│     └── Automatic restart on failure                        │
│                                                             │
│  3. Logging (journald)                                      │
│     ├── Centralized logging                                 │
│     ├── Structured log entries                              │
│     └── Log rotation and management                         │
│                                                             │
│  4. Login Management (logind)                               │
│     ├── User sessions                                       │
│     ├── Seat management                                     │
│     └── Power management                                    │
│                                                             │
│  5. Network Management (networkd - optional)                │
│  6. Time Synchronization (timesyncd - optional)             │
│  7. Name Resolution (resolved - optional)                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Systemd Units

Systemd manages "units" - resources it knows how to manage. There are several types:

```
Systemd Unit Types:
┌────────────────┬───────────────────────────────────────────────┐
│ Unit Type      │ Purpose                                       │
├────────────────┼───────────────────────────────────────────────┤
│ .service       │ System services (daemons)                     │
│ .socket        │ Socket-based activation                       │
│ .target        │ Group of units (like runlevels)               │
│ .mount         │ Filesystem mount points                       │
│ .automount     │ On-demand mount points                        │
│ .swap          │ Swap space                                    │
│ .path          │ Path-based activation                         │
│ .timer         │ Timer-based activation (like cron)            │
│ .device        │ Device units                                  │
│ .slice         │ Resource management (cgroups)                 │
│ .scope         │ Externally created processes                  │
└────────────────┴───────────────────────────────────────────────┘
```

### Unit File Locations

```
Unit File Locations (in order of precedence):
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  /etc/systemd/system/                                       │
│  ├── Highest priority                                       │
│  ├── Local customizations and overrides                     │
│  ├── Administrator-created units                            │
│  └── USE THIS for your custom services                      │
│                                                             │
│  /run/systemd/system/                                       │
│  ├── Runtime units                                          │
│  └── Generated at runtime, cleared on reboot                │
│                                                             │
│  /lib/systemd/system/ (or /usr/lib/systemd/system/)        │
│  ├── Lowest priority                                        │
│  ├── Package-provided units                                 │
│  └── DO NOT modify these directly                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 4.2 Service Management with systemctl

```bash
# systemctl - Control the systemd system and service manager

# Service status
systemctl status nginx                # Detailed status
systemctl is-active nginx            # Just check if running
systemctl is-enabled nginx           # Check if starts at boot
systemctl is-failed nginx            # Check if failed

# Understanding status output:
# ● nginx.service - A high performance web server
#    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
#    Active: active (running) since Mon 2024-01-15 10:00:00 UTC; 2h ago
#      Docs: man:nginx(8)
#  Main PID: 1234 (nginx)
#    Status: "Accepting connections"
#     Tasks: 3 (limit: 4915)
#    Memory: 10.0M
#    CGroup: /system.slice/nginx.service
#            ├─1234 nginx: master process /usr/sbin/nginx
#            ├─1235 nginx: worker process
#            └─1236 nginx: worker process
#
# Logs from journald follow...

# Start/Stop/Restart
systemctl start nginx                # Start service
systemctl stop nginx                 # Stop service
systemctl restart nginx              # Stop then start
systemctl reload nginx               # Reload config without restart
systemctl reload-or-restart nginx    # Reload if supported, else restart
systemctl try-restart nginx          # Restart only if running

# Enable/Disable (boot behavior)
systemctl enable nginx               # Start at boot
systemctl disable nginx              # Don't start at boot
systemctl enable --now nginx         # Enable AND start now
systemctl disable --now nginx        # Disable AND stop now
systemctl reenable nginx             # Disable then enable (reset symlinks)

# Mask/Unmask (prevent starting)
systemctl mask nginx                 # Prevent service from starting at all
systemctl unmask nginx               # Allow starting again
# Masking creates symlink to /dev/null
# Even 'start' won't work on masked service

# List units
systemctl list-units                        # All active units
systemctl list-units --all                 # All units (including inactive)
systemctl list-units --type=service        # Only services
systemctl list-units --state=failed        # Failed units only
systemctl list-unit-files                  # All installed units
systemctl list-unit-files --type=service   # All installed services

# Dependencies
systemctl list-dependencies nginx          # What nginx needs
systemctl list-dependencies --reverse nginx # What needs nginx
systemctl list-dependencies --all nginx    # Recursive

# Unit file operations
systemctl cat nginx                        # Show unit file contents
systemctl edit nginx                       # Create override file
systemctl edit --full nginx               # Edit full unit file
systemctl daemon-reload                   # Reload after editing unit files
systemctl show nginx                      # Show all properties
systemctl show nginx -p MainPID           # Show specific property

# System commands
systemctl reboot                           # Reboot system
systemctl poweroff                         # Power off
systemctl suspend                          # Suspend
systemctl hibernate                        # Hibernate
systemctl rescue                           # Single-user mode
systemctl emergency                        # Emergency mode
systemctl default                          # Return to normal mode
systemctl get-default                      # Show default target
systemctl set-default multi-user.target   # Set default target
systemctl isolate multi-user.target       # Switch to target immediately
```

## 4.3 Creating Custom Service Units

### Basic Service Unit Structure

```ini
# /etc/systemd/system/myapp.service

[Unit]
# Description and dependencies
Description=My Application
Documentation=https://example.com/docs
After=network.target postgresql.service
Wants=postgresql.service
Requires=network-online.target
Before=nginx.service

[Service]
# Service configuration
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
EnvironmentFile=/etc/myapp/environment
ExecStart=/opt/myapp/bin/start.sh
ExecStop=/opt/myapp/bin/stop.sh
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10
TimeoutStartSec=30
TimeoutStopSec=30

[Install]
# Install configuration
WantedBy=multi-user.target
```

### [Unit] Section Explained

```ini
[Unit]
# Human-readable description
Description=My Application Server

# Link to documentation
Documentation=https://example.com/docs
Documentation=man:myapp(8)

# Ordering dependencies (doesn't imply requirement)
After=network.target              # Start after these units
Before=nginx.service              # Start before these units

# Requirement dependencies
Requires=postgresql.service       # MUST be running (fails if not)
Wants=redis.service               # Should be running (soft dependency)
BindsTo=other.service             # Like Requires, but also stops together

# Conflicts (cannot run together)
Conflicts=other.service

# Conditions (skip if not met, don't fail)
ConditionPathExists=/opt/myapp/config.yaml
ConditionPathIsDirectory=/opt/myapp
ConditionFileIsExecutable=/opt/myapp/bin/start.sh
ConditionHost=production-server
ConditionVirtualization=!container    # Don't run in container

# Assertions (fail if not met)
AssertPathExists=/opt/myapp/bin/start.sh
```

### [Service] Section Explained

```ini
[Service]
# Service type (how systemd tracks process)
Type=simple                       # Default, ExecStart is main process
# Type=forking                    # Forks to background, use PIDFile=
# Type=oneshot                    # Runs once then exits
# Type=notify                     # Sends notification when ready
# Type=dbus                       # D-Bus activation
# Type=idle                       # Delayed until all jobs finished

# Execution
User=myapp                        # Run as this user
Group=myapp                       # Run as this group
WorkingDirectory=/opt/myapp       # Set working directory
RootDirectory=/opt/chroot         # Chroot (optional)

# Environment
Environment=NODE_ENV=production
Environment="PATH=/opt/myapp/bin:/usr/bin"
EnvironmentFile=/etc/myapp/env    # Read from file
EnvironmentFile=-/etc/myapp/optional.env  # - means optional

# Commands
ExecStartPre=/opt/myapp/bin/pre-start.sh   # Run before ExecStart
ExecStart=/opt/myapp/bin/start.sh          # Main command (required)
ExecStartPost=/opt/myapp/bin/post-start.sh # Run after ExecStart
ExecStop=/opt/myapp/bin/stop.sh            # Stop command
ExecReload=/bin/kill -HUP $MAINPID         # Reload command

# For forking type
PIDFile=/run/myapp.pid

# Restart behavior
Restart=always                    # Always restart when stopped
# Restart=no                      # Never restart
# Restart=on-success              # Only on clean exit
# Restart=on-failure              # Only on unclean exit
# Restart=on-abnormal             # On signal, timeout, watchdog
# Restart=on-abort                # Only on signal
# Restart=on-watchdog             # Only on watchdog timeout

RestartSec=10                     # Wait 10 seconds before restart
RestartPreventExitStatus=0 SIGTERM # Don't restart on these
StartLimitIntervalSec=300         # Time window for start limit
StartLimitBurst=5                 # Max starts in window

# Timeouts
TimeoutStartSec=30                # Timeout for starting
TimeoutStopSec=30                 # Timeout for stopping
TimeoutSec=30                     # Both start and stop
WatchdogSec=30                    # Watchdog ping interval

# Resource limits
LimitNOFILE=65535                 # Max open files
LimitNPROC=4096                   # Max processes
LimitCORE=infinity                # Core dump size
MemoryLimit=2G                    # Memory limit

# Security
NoNewPrivileges=yes               # Can't gain privileges
ProtectSystem=full                # Read-only /usr, /boot
ProtectHome=yes                   # No access to /home
PrivateTmp=yes                    # Isolated /tmp
ReadOnlyPaths=/etc
ReadWritePaths=/var/lib/myapp
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Standard I/O
StandardOutput=journal            # Log to journal
StandardError=journal
StandardInput=null
SyslogIdentifier=myapp
```

### [Install] Section Explained

```ini
[Install]
# Target to enable this unit for
WantedBy=multi-user.target       # Normal multi-user system
# WantedBy=graphical.target      # Graphical environment

# Required by (stronger than WantedBy)
RequiredBy=some-other.service

# Aliases
Alias=myapp.service
```

### Common Service Patterns

```ini
# Simple daemon
[Unit]
Description=Simple Daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/mydaemon
Restart=always

[Install]
WantedBy=multi-user.target

# Forking daemon (traditional)
[Unit]
Description=Forking Daemon
After=network.target

[Service]
Type=forking
PIDFile=/run/mydaemon.pid
ExecStart=/usr/bin/mydaemon --daemon
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target

# One-shot task
[Unit]
Description=Run once at boot
After=network.target

[Service]
Type=oneshot
ExecStart=/opt/scripts/bootstrap.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

# Node.js application
[Unit]
Description=Node.js App
After=network.target

[Service]
Type=simple
User=nodejs
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# Java application
[Unit]
Description=Java Application
After=network.target

[Service]
Type=simple
User=java
ExecStart=/usr/bin/java -Xmx2g -jar /opt/myapp/app.jar
Restart=always
RestartSec=10
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

### Overriding Unit Files

```bash
# Create override directory
systemctl edit nginx              # Opens editor

# This creates /etc/systemd/system/nginx.service.d/override.conf
# Add only the settings you want to change:

[Service]
Environment=NGINX_WORKERS=4
LimitNOFILE=65535

# Reload after changes
systemctl daemon-reload
systemctl restart nginx

# View effective configuration
systemctl cat nginx              # Shows combined config
```

## 4.4 Journald and Logging

Journald is systemd's logging service. It collects and manages journal entries from various sources.

```bash
# journalctl - Query the systemd journal

# Basic usage
journalctl                        # All journal entries
journalctl -e                     # Jump to end
journalctl -f                     # Follow (like tail -f)

# Filter by unit
journalctl -u nginx               # Nginx logs only
journalctl -u nginx -u php-fpm    # Multiple units
journalctl -u nginx --since today

# Filter by time
journalctl --since "2024-01-15"
journalctl --since "2024-01-15 10:00:00"
journalctl --since "1 hour ago"
journalctl --since today
journalctl --since yesterday --until today
journalctl -b                     # Current boot only
journalctl -b -1                  # Previous boot
journalctl --list-boots           # List all boots

# Filter by priority
journalctl -p err                 # Error and above
journalctl -p warning             # Warning and above
journalctl -p 0..3                # emerg, alert, crit, err
# Priority levels: emerg(0), alert(1), crit(2), err(3), 
#                  warning(4), notice(5), info(6), debug(7)

# Filter by other criteria
journalctl _COMM=sudo             # Command name
journalctl _PID=1234              # Process ID
journalctl _UID=1000              # User ID
journalctl _GID=1000              # Group ID
journalctl _SYSTEMD_UNIT=nginx.service
journalctl _TRANSPORT=kernel      # Kernel messages
journalctl CONTAINER_NAME=mycontainer

# Output format
journalctl -o verbose             # All fields
journalctl -o json                # JSON format
journalctl -o json-pretty         # Pretty JSON
journalctl -o short-iso           # ISO timestamps
journalctl -o cat                 # No metadata

# Other options
journalctl -n 100                 # Last 100 entries
journalctl -x                     # Add explanatory text
journalctl --no-pager             # No pagination
journalctl --disk-usage           # Journal disk usage
journalctl --vacuum-size=500M     # Reduce to 500MB
journalctl --vacuum-time=1month   # Remove older than 1 month

# Combining filters
journalctl -u nginx -p err --since today -f
```

### Journald Configuration

```bash
# /etc/systemd/journald.conf

[Journal]
Storage=persistent              # auto, volatile, persistent, none
Compress=yes                    # Compress journal files
SplitMode=uid                   # Split by UID
RateLimitIntervalSec=30s       # Rate limiting
RateLimitBurst=10000
SystemMaxUse=500M              # Max journal size
SystemKeepFree=1G              # Keep this much free
MaxFileSec=1month              # Rotate after this time
MaxRetentionSec=1year          # Keep for this long

# Apply changes
systemctl restart systemd-journald
```

## 4.5 Systemd Timers (Cron Alternative)

Systemd timers are the modern replacement for cron jobs.

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.sh
User=backup

# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=*-*-* 02:00:00      # Every day at 2 AM
# OnCalendar=daily             # Alternative: daily at midnight
# OnCalendar=weekly            # Weekly (Monday midnight)
# OnCalendar=Mon *-*-* 02:00   # Every Monday at 2 AM
# OnCalendar=*-*-01 00:00:00   # First day of month
# OnCalendar=*:0/15            # Every 15 minutes

# Time after boot
OnBootSec=5min                 # 5 minutes after boot
OnUnitActiveSec=1h             # 1 hour after last activation

Persistent=yes                 # Run if missed (e.g., system was off)
RandomizedDelaySec=30min       # Random delay up to 30 min
AccuracySec=1s                 # Timer accuracy

[Install]
WantedBy=timers.target
```

```bash
# Enable timer
systemctl enable --now backup.timer

# List timers
systemctl list-timers
systemctl list-timers --all

# Check timer status
systemctl status backup.timer
systemctl status backup.service

# Test run immediately
systemctl start backup.service

# Timer calendar expressions:
# *-*-* 00:00:00      : Every day at midnight
# Mon *-*-* 00:00:00  : Every Monday at midnight
# *-*-01 00:00:00     : First of every month
# *-01-01 00:00:00    : January 1st every year
# *:*:00              : Every minute
# *:00:00             : Every hour
# *:0/15              : Every 15 minutes

# Test calendar expression
systemd-analyze calendar "*-*-* 02:00:00"
systemd-analyze calendar "Mon *-*-* 00:00:00"
```

---

# Chapter 5: Package Management

## 5.1 Debian/Ubuntu Package Management (APT)

APT (Advanced Package Tool) is the package management system for Debian-based distributions.

### How APT Works

```
APT Architecture:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Package Repositories                                       │
│  (mirror.ubuntu.com, archive.ubuntu.com)                   │
│      │                                                      │
│      │  HTTP/HTTPS                                          │
│      ▼                                                      │
│  ┌──────────────────────────────────────────────┐          │
│  │  /var/lib/apt/lists/                          │          │
│  │  (Local package metadata cache)               │          │
│  └────────────────────┬─────────────────────────┘          │
│                       │                                     │
│                       ▼                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │  apt / apt-get                                │          │
│  │  (High-level package manager)                 │          │
│  └────────────────────┬─────────────────────────┘          │
│                       │                                     │
│                       ▼                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │  dpkg                                         │          │
│  │  (Low-level package installer)               │          │
│  └────────────────────┬─────────────────────────┘          │
│                       │                                     │
│                       ▼                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │  /var/lib/dpkg/                               │          │
│  │  (Installed package database)                 │          │
│  └──────────────────────────────────────────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### APT Commands

```bash
# apt - Modern command (recommended)
apt update                     # Update package lists
apt upgrade                    # Upgrade installed packages
apt full-upgrade               # Upgrade with dependency changes
apt install package            # Install package
apt install package=version    # Install specific version
apt install ./package.deb      # Install local .deb file
apt remove package             # Remove package (keep config)
apt purge package              # Remove package and config
apt autoremove                 # Remove unused dependencies
apt search keyword             # Search packages
apt show package               # Show package info
apt list --installed           # List installed packages
apt list --upgradable          # List upgradable packages
apt edit-sources               # Edit sources list
apt clean                      # Clear package cache
apt autoclean                  # Clear old package cache

# apt-get - Traditional command (for scripts)
apt-get update
apt-get upgrade
apt-get dist-upgrade           # Like full-upgrade
apt-get install package
apt-get remove package
apt-get purge package
apt-get autoremove
apt-get clean
apt-get autoclean
apt-get download package       # Download without installing
apt-get source package         # Download source package

# apt-cache - Query package cache
apt-cache search keyword       # Search packages
apt-cache show package         # Show package details
apt-cache policy package       # Show version/source info
apt-cache depends package      # Show dependencies
apt-cache rdepends package     # Show reverse dependencies
apt-cache madison package      # Show available versions

# Options for apt/apt-get
apt install -y package         # Yes to all prompts
apt install --no-install-recommends package  # Minimal install
apt install --reinstall package # Force reinstall
apt install --dry-run package  # Simulate installation
apt install -d package         # Download only
apt install package-           # Remove package (note the minus)
apt install package+           # Install package (note the plus)

# Hold/Unhold packages (prevent upgrades)
apt-mark hold package
apt-mark unhold package
apt-mark showhold

# dpkg - Low-level package manager
dpkg -i package.deb            # Install .deb file
dpkg -r package                # Remove package
dpkg -P package                # Purge package
dpkg -l                        # List installed packages
dpkg -l | grep keyword         # Search installed
dpkg -L package                # List files in package
dpkg -S /path/to/file          # Which package owns file
dpkg -s package                # Package status
dpkg --configure -a            # Configure pending packages
dpkg --audit                   # Check for problems

# Fix broken installations
apt --fix-broken install
dpkg --configure -a
apt update && apt upgrade
```

### APT Repository Configuration

```bash
# Main sources file
/etc/apt/sources.list

# Additional sources (recommended for third-party)
/etc/apt/sources.list.d/*.list

# Source line format:
# deb [options] URL distribution components
# deb-src [options] URL distribution components

# Example /etc/apt/sources.list for Ubuntu 22.04:
deb http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse

# Components:
# main       : Officially supported free software
# restricted : Officially supported non-free software
# universe   : Community-maintained free software
# multiverse : Non-free software

# Add third-party repository
# 1. Add GPG key
curl -fsSL https://example.com/key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/example.gpg

# 2. Add repository
echo "deb [signed-by=/etc/apt/keyrings/example.gpg] https://example.com/repo stable main" | \
    sudo tee /etc/apt/sources.list.d/example.list

# 3. Update and install
sudo apt update
sudo apt install package

# add-apt-repository (simpler for PPAs)
sudo add-apt-repository ppa:user/ppa-name
sudo apt update
sudo apt install package

# Remove PPA
sudo add-apt-repository --remove ppa:user/ppa-name
```

## 5.2 RHEL/CentOS Package Management (YUM/DNF)

DNF (Dandified YUM) is the modern package manager for RHEL-based distributions. YUM is the older version but commands are mostly compatible.

```bash
# DNF/YUM commands
dnf check-update               # Check for updates
dnf upgrade                    # Upgrade all packages
dnf install package            # Install package
dnf install package-version    # Install specific version
dnf install ./package.rpm      # Install local RPM
dnf remove package             # Remove package
dnf autoremove                 # Remove unused dependencies
dnf search keyword             # Search packages
dnf info package               # Show package info
dnf list installed             # List installed packages
dnf list available             # List available packages
dnf list updates               # List available updates
dnf provides /path/to/file     # Which package provides file
dnf repolist                   # List enabled repos
dnf clean all                  # Clear cache
dnf history                    # Show transaction history
dnf history undo 15            # Undo transaction 15

# Group operations
dnf group list                 # List available groups
dnf group install "Development Tools"
dnf group remove "Development Tools"

# RPM - Low-level package manager
rpm -ivh package.rpm           # Install package
rpm -Uvh package.rpm           # Upgrade package
rpm -e package                 # Remove package
rpm -qa                        # List all installed
rpm -qi package                # Package info
rpm -ql package                # List files in package
rpm -qf /path/to/file          # Which package owns file
rpm -qc package                # List config files
rpm -V package                 # Verify package

# Repository configuration
/etc/yum.repos.d/*.repo

# Example repo file:
[myrepo]
name=My Repository
baseurl=https://example.com/repo/
enabled=1
gpgcheck=1
gpgkey=https://example.com/key.gpg
```

---

# Chapter 6: Storage and Filesystems

## 6.1 Disk Partitioning

### Understanding Disk Naming

```
Linux Disk Naming:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  /dev/sda    : First SATA/SCSI/USB disk                    │
│  /dev/sdb    : Second SATA/SCSI/USB disk                   │
│  /dev/sda1   : First partition on sda                       │
│  /dev/sda2   : Second partition on sda                      │
│                                                             │
│  /dev/nvme0n1    : First NVMe disk                          │
│  /dev/nvme0n1p1  : First partition on NVMe disk             │
│  /dev/nvme1n1    : Second NVMe disk                         │
│                                                             │
│  /dev/vda    : First VirtIO disk (VMs)                      │
│  /dev/xvda   : First Xen virtual disk (AWS)                 │
│                                                             │
│  /dev/mmcblk0    : First SD card / eMMC                     │
│  /dev/mmcblk0p1  : First partition on SD card               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Partition Tables: MBR vs GPT

```
MBR (Master Boot Record):
├── Legacy partitioning scheme
├── Maximum disk size: 2 TB
├── Maximum 4 primary partitions
├── Can use extended partition for more (logical partitions)
├── BIOS boot compatible
└── Being phased out in favor of GPT

GPT (GUID Partition Table):
├── Modern partitioning scheme
├── Maximum disk size: 9.4 ZB (practically unlimited)
├── Up to 128 partitions (by default)
├── Each partition has unique GUID
├── UEFI boot compatible
├── Backup partition table at end of disk
└── Recommended for all new installations
```

### Partitioning Commands

```bash
# View disk information
lsblk                          # List block devices
lsblk -f                       # Show filesystems
fdisk -l                       # List all disks and partitions
parted -l                      # List with GPT support
blkid                          # Show UUIDs and labels

# fdisk - MBR partitioning
sudo fdisk /dev/sda
# Interactive commands:
#   p : Print partition table
#   n : New partition
#   d : Delete partition
#   t : Change partition type
#   w : Write and exit
#   q : Quit without saving

# gdisk - GPT partitioning
sudo gdisk /dev/sda
# Similar commands to fdisk, but for GPT

# parted - Supports both MBR and GPT
sudo parted /dev/sda
# Interactive commands:
#   print        : Print partition table
#   mklabel gpt  : Create GPT partition table
#   mkpart       : Create partition
#   rm NUMBER    : Remove partition
#   resizepart   : Resize partition
#   quit         : Exit

# Non-interactive parted
sudo parted /dev/sda mklabel gpt
sudo parted /dev/sda mkpart primary ext4 1MiB 10GiB
sudo parted /dev/sda mkpart primary ext4 10GiB 100%
```

## 6.2 Filesystems

### Common Filesystem Types

```
Linux Filesystems:
┌────────────┬───────────────────────────────────────────────┐
│ Filesystem │ Description                                   │
├────────────┼───────────────────────────────────────────────┤
│ ext4       │ Standard Linux filesystem, mature and stable  │
│            │ Max file: 16 TB, Max volume: 1 EB             │
│            │ Journaling, extents, online resize            │
├────────────┼───────────────────────────────────────────────┤
│ XFS        │ High-performance, good for large files        │
│            │ Max file: 8 EB, Max volume: 8 EB              │
│            │ Default for RHEL/CentOS, parallel I/O         │
├────────────┼───────────────────────────────────────────────┤
│ Btrfs      │ Copy-on-write, snapshots, checksums           │
│            │ Compression, RAID, subvolumes                 │
│            │ Default for some distros (openSUSE, Fedora)   │
├────────────┼───────────────────────────────────────────────┤
│ ZFS        │ Advanced copy-on-write filesystem             │
│            │ Snapshots, compression, deduplication         │
│            │ Not in mainline kernel (licensing)            │
├────────────┼───────────────────────────────────────────────┤
│ tmpfs      │ RAM-based filesystem                          │
│            │ Used for /tmp, /run                           │
│            │ Data lost on reboot                           │
├────────────┼───────────────────────────────────────────────┤
│ vfat/FAT32 │ Windows compatible, USB drives                │
│            │ No permissions, 4GB file limit                │
├────────────┼───────────────────────────────────────────────┤
│ exFAT      │ Modern FAT, large file support                │
│            │ Good for cross-platform USB drives            │
├────────────┼───────────────────────────────────────────────┤
│ NTFS       │ Windows native, read/write with ntfs-3g       │
└────────────┴───────────────────────────────────────────────┘
```

### Creating Filesystems

```bash
# Create filesystems
mkfs.ext4 /dev/sda1            # Create ext4
mkfs.xfs /dev/sda1             # Create XFS
mkfs.btrfs /dev/sda1           # Create Btrfs
mkfs.vfat /dev/sda1            # Create FAT32

# With options
mkfs.ext4 -L "mylabel" /dev/sda1          # With label
mkfs.ext4 -m 1 /dev/sda1                  # 1% reserved (vs default 5%)
mkfs.xfs -L "mylabel" /dev/sda1
mkfs.xfs -f /dev/sda1                     # Force (overwrite)

# Tune existing filesystem
tune2fs -L "newlabel" /dev/sda1           # Change label (ext)
tune2fs -m 1 /dev/sda1                    # Change reserved space
tune2fs -l /dev/sda1                      # Show filesystem info
xfs_admin -L "newlabel" /dev/sda1         # Change label (XFS)
```

### Mounting Filesystems

```bash
# Manual mounting
mount /dev/sda1 /mnt                      # Basic mount
mount -t ext4 /dev/sda1 /mnt              # Specify type
mount -o ro /dev/sda1 /mnt                # Read-only
mount -o noexec /dev/sda1 /mnt            # No execute
mount UUID=abc-123 /mnt                   # Mount by UUID
mount LABEL=mylabel /mnt                  # Mount by label

# Mount options
mount -o rw,noexec,nosuid,nodev /dev/sda1 /mnt
# rw       : Read-write
# ro       : Read-only
# noexec   : Cannot execute binaries
# nosuid   : Ignore SUID bits
# nodev    : Ignore device files
# noatime  : Don't update access time (performance)
# nodiratime: Don't update directory access time
# relatime : Update atime less frequently (default)
# sync     : Synchronous I/O
# async    : Asynchronous I/O (default)
# defaults : rw, suid, dev, exec, auto, nouser, async

# Unmounting
umount /mnt                               # Unmount by mount point
umount /dev/sda1                          # Unmount by device
umount -l /mnt                            # Lazy unmount (when free)
umount -f /mnt                            # Force unmount (NFS)

# Check what's using mount point
fuser -m /mnt                             # List processes
lsof /mnt                                 # Detailed list
```

### Persistent Mounts (/etc/fstab)

```bash
# /etc/fstab format:
# <filesystem>  <mount point>  <type>  <options>  <dump>  <pass>

# Examples:
# Root filesystem
UUID=abc-123-def  /              ext4    defaults        0  1

# Home directory
UUID=xyz-789-uvw  /home          ext4    defaults        0  2

# Swap
UUID=swap-uuid    none           swap    sw              0  0

# Data disk
/dev/sdb1         /data          xfs     defaults,noatime 0  2

# NFS mount
server:/share     /nfs           nfs     defaults,_netdev 0  0

# CIFS/SMB mount
//server/share    /smb           cifs    credentials=/etc/cifs-creds,uid=1000 0 0

# tmpfs
tmpfs             /tmp           tmpfs   defaults,size=2G 0  0

# Bind mount (mount directory to another location)
/source/dir       /target/dir    none    bind            0  0

# Mount all from fstab
mount -a

# Check fstab for errors (before reboot!)
mount -a
# Or safer:
findmnt --verify

# Fields explained:
# dump (5th field): 0 = no dump, 1 = dump (backup)
# pass (6th field): 
#   0 = no fsck
#   1 = fsck first (root)
#   2 = fsck after root
```

## 6.3 Logical Volume Management (LVM)

LVM provides flexible disk management with features like resizing, snapshots, and spanning multiple disks.

```
LVM Architecture:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Logical Volumes (LVs)                   │   │
│  │   /dev/vg_data/lv_home    /dev/vg_data/lv_var       │   │
│  │         100GB                    50GB                │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                  │
│  ┌───────────────────────▼─────────────────────────────┐   │
│  │               Volume Group (VG)                      │   │
│  │                   vg_data                            │   │
│  │                    200GB                             │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                  │
│  ┌───────────┬───────────┴───────────┬─────────────────┐   │
│  │           │                       │                  │   │
│  ▼           ▼                       ▼                  │   │
│ ┌────────┐ ┌────────┐            ┌────────┐            │   │
│ │  PV    │ │  PV    │            │  PV    │            │   │
│ │/dev/sda│ │/dev/sdb│            │/dev/sdc│            │   │
│ │ 100GB  │ │ 50GB   │            │ 50GB   │            │   │
│ └────────┘ └────────┘            └────────┘            │   │
│                                                             │
│  Physical Volumes (PVs) - Actual disks or partitions       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### LVM Commands

```bash
# Physical Volumes (PV)
pvcreate /dev/sdb                  # Create PV
pvdisplay                          # Display PVs
pvs                                # Summary view
pvscan                             # Scan for PVs
pvremove /dev/sdb                  # Remove PV

# Volume Groups (VG)
vgcreate vg_data /dev/sdb /dev/sdc # Create VG
vgdisplay                          # Display VGs
vgs                                # Summary view
vgextend vg_data /dev/sdd          # Add disk to VG
vgreduce vg_data /dev/sdd          # Remove disk from VG
vgremove vg_data                   # Remove VG

# Logical Volumes (LV)
lvcreate -L 10G -n lv_home vg_data      # Create 10GB LV
lvcreate -l 100%FREE -n lv_data vg_data # Use all free space
lvcreate -l 50%VG -n lv_data vg_data    # Use 50% of VG
lvdisplay                          # Display LVs
lvs                                # Summary view

# Create filesystem
mkfs.ext4 /dev/vg_data/lv_home
mount /dev/vg_data/lv_home /home

# Extend LV (ONLINE!)
lvextend -L +10G /dev/vg_data/lv_home     # Add 10GB
lvextend -L 50G /dev/vg_data/lv_home      # Extend to 50GB
lvextend -l +100%FREE /dev/vg_data/lv_home # Use all free

# Extend filesystem (ONLINE!)
resize2fs /dev/vg_data/lv_home            # ext4
xfs_growfs /home                          # XFS (specify mount point)

# Combine extend and resize
lvextend -r -L +10G /dev/vg_data/lv_home  # -r resizes filesystem too

# Reduce LV (OFFLINE, DANGEROUS!)
umount /home
e2fsck -f /dev/vg_data/lv_home            # Check filesystem
resize2fs /dev/vg_data/lv_home 30G        # Shrink filesystem first
lvreduce -L 30G /dev/vg_data/lv_home      # Then shrink LV
mount /dev/vg_data/lv_home /home

# Remove LV
umount /home
lvremove /dev/vg_data/lv_home

# Snapshots
lvcreate -L 5G -s -n snap_home /dev/vg_data/lv_home  # Create snapshot
lvs                                        # View snapshots
mount /dev/vg_data/snap_home /mnt/snapshot # Mount snapshot
lvremove /dev/vg_data/snap_home            # Remove snapshot

# Rename
lvrename vg_data lv_old lv_new
vgrename vg_old vg_new
```

## 6.4 Disk Usage and Monitoring

```bash
# df - Disk Free space
df                                 # All mounted filesystems
df -h                              # Human-readable
df -hT                             # Include filesystem type
df -i                              # Inode usage
df /path                           # Specific filesystem

# du - Disk Usage by directory
du -h /path                        # Size of directory tree
du -sh /path                       # Summary only
du -sh /path/*                     # Size of each item
du -h --max-depth=1 /path          # One level deep
du -sh /var/log/* | sort -h        # Sorted by size

# Find large files and directories
du -sh /var/* | sort -h | tail -10     # Top 10 directories
find / -type f -size +100M 2>/dev/null | head -20
find / -xdev -type f -size +100M       # Don't cross filesystems

# ncdu - Interactive disk usage (install separately)
ncdu /path

# Check filesystem
fsck /dev/sda1                     # Check and repair (UNMOUNTED!)
fsck -n /dev/sda1                  # Check only, no repair
fsck -y /dev/sda1                  # Yes to all repairs
e2fsck -f /dev/sda1                # Force check (ext)
xfs_repair /dev/sda1               # Repair XFS (unmounted)

# Disk I/O monitoring
iostat                             # Basic I/O stats
iostat -x 1                        # Extended stats, 1 second interval
iotop                              # I/O by process (install separately)

# SMART monitoring (disk health)
smartctl -a /dev/sda               # All SMART data
smartctl -H /dev/sda               # Health status
smartctl -t short /dev/sda         # Run short test
smartctl -t long /dev/sda          # Run long test
```

---

# Chapter 7: Performance Analysis and Tuning

## 7.1 System Performance Overview

```bash
# Quick system overview
uptime                             # Load average
top                                # Interactive process viewer
htop                               # Enhanced process viewer
vmstat 1                           # System statistics every second
dstat                              # Versatile resource stats

# Comprehensive monitoring
sar                                # System Activity Reporter
glances                            # All-in-one monitoring

# Load average explained:
# uptime output: load average: 0.50, 0.75, 1.00
# Three values: 1-minute, 5-minute, 15-minute averages
#
# Load = number of processes wanting CPU time
# Load of 1.0 on single CPU = fully utilized
# Load of 2.0 on single CPU = overloaded by 2x
# Load of 4.0 on 4-CPU system = fully utilized
#
# Rule of thumb: load should be < number of CPUs
# Check CPU count: nproc or lscpu
```

## 7.2 CPU Analysis

```bash
# CPU information
lscpu                              # CPU architecture info
cat /proc/cpuinfo                  # Detailed CPU info
nproc                              # Number of processors

# CPU usage
mpstat 1                           # Per-CPU stats every second
mpstat -P ALL 1                    # All CPUs individually
pidstat 1                          # Per-process CPU usage

# Top CPU consumers
ps aux --sort=-%cpu | head -10
top -b -n1 | head -20

# CPU profiling
perf top                           # Live CPU profiling
perf record -g command             # Record CPU profile
perf report                        # View profile

# Understanding CPU metrics:
# %user   : Time in user space (applications)
# %system : Time in kernel space (system calls)
# %iowait : Time waiting for I/O
# %idle   : Time doing nothing
# %steal  : Time stolen by hypervisor (virtualization)
#
# High %user    : Application doing heavy computation
# High %system  : Many system calls, context switches
# High %iowait  : Disk bottleneck
# High %steal   : VM host overloaded
```

## 7.3 Memory Analysis

```bash
# Memory information
free -h                            # Memory summary
cat /proc/meminfo                  # Detailed memory info
vmstat 1                           # Virtual memory stats

# Understanding free output:
#               total    used    free  shared  buff/cache  available
# Mem:           16G      4G      8G    100M         4G        11G
#
# total     : Physical RAM
# used      : Currently in use (includes buffers/cache)
# free      : Not used at all
# shared    : Used by tmpfs
# buff/cache: Used for disk buffers and page cache
# available : Memory available for applications
#
# IMPORTANT: Linux uses free memory for caching
# "available" is more useful than "free"

# Memory consumers
ps aux --sort=-%mem | head -10     # Top memory consumers
smem                               # Memory reporting tool
pmap -x <pid>                      # Process memory map

# Memory pressure
cat /proc/meminfo | grep -i swap
vmstat 1 | awk '{print $7,$8}'     # si/so (swap in/out)
# If si/so are high, system is swapping (memory pressure)

# Clear caches (testing only)
sync; echo 3 > /proc/sys/vm/drop_caches
# 1 = page cache, 2 = dentries/inodes, 3 = both
```

## 7.4 I/O Analysis

```bash
# I/O statistics
iostat -x 1                        # Extended I/O stats
iotop                              # I/O by process
pidstat -d 1                       # Per-process I/O

# Understanding iostat:
# Device:  rrqm/s wrqm/s  r/s   w/s  rkB/s wkB/s  await  %util
# sda      0.00   10.00  5.00 20.00 100.00 500.00  10.00  25.00
#
# rrqm/s, wrqm/s : Read/write requests merged per second
# r/s, w/s       : Read/write operations per second
# rkB/s, wkB/s   : Read/write KB per second
# await          : Average time for requests (ms)
# %util          : Percentage of time device busy
#
# High await    : Slow disk or overloaded
# High %util    : Disk is bottleneck

# Block trace
blktrace /dev/sda -w 30            # Trace I/O for 30 seconds
blkparse sda                       # Parse trace

# Check for I/O wait
top                                # Watch %wa
vmstat 1                           # Watch wa column
```

## 7.5 Network Performance

```bash
# Network statistics
ss -s                              # Socket summary
netstat -s                         # Detailed network stats
ip -s link                         # Interface statistics
sar -n DEV 1                       # Network interface stats

# Bandwidth monitoring
iftop                              # Interface bandwidth
nethogs                            # Bandwidth by process
vnstat                             # Traffic history

# Connection analysis
ss -tuln                           # Listening ports
ss -tun                            # Active connections
ss -tunp                           # With process info
ss -s                              # Statistics

# Network troubleshooting
ping host                          # Connectivity
traceroute host                    # Path analysis
mtr host                           # Combined ping/traceroute
tcpdump -i eth0                    # Packet capture

# Network performance testing
iperf3 -s                          # Server mode
iperf3 -c server                   # Client mode
```

## 7.6 Kernel Tuning (sysctl)

```bash
# View kernel parameters
sysctl -a                          # All parameters
sysctl vm.swappiness               # Specific parameter
cat /proc/sys/vm/swappiness        # Direct read

# Set parameters (temporary)
sysctl -w vm.swappiness=10
echo 10 > /proc/sys/vm/swappiness

# Set parameters (permanent)
# Add to /etc/sysctl.conf or /etc/sysctl.d/*.conf
vm.swappiness = 10
net.core.somaxconn = 65535

# Apply changes
sysctl -p                          # Load /etc/sysctl.conf
sysctl -p /etc/sysctl.d/custom.conf

# Common tuning parameters:
# Memory
vm.swappiness = 10                 # Reduce swap usage (0-100)
vm.dirty_ratio = 20                # % of memory for dirty pages
vm.dirty_background_ratio = 5      # % when background writeback starts

# Network
net.core.somaxconn = 65535                    # Max socket connections
net.ipv4.tcp_max_syn_backlog = 65535          # SYN queue size
net.core.netdev_max_backlog = 65535           # Network device queue
net.ipv4.ip_local_port_range = 1024 65535     # Ephemeral port range
net.ipv4.tcp_fin_timeout = 30                 # TIME_WAIT timeout
net.ipv4.tcp_tw_reuse = 1                     # Reuse TIME_WAIT sockets

# File handles
fs.file-max = 2097152                         # Max open files system-wide
fs.inotify.max_user_watches = 524288          # Max inotify watches

# For high-performance servers
net.core.rmem_max = 16777216                  # Max receive buffer
net.core.wmem_max = 16777216                  # Max send buffer
net.ipv4.tcp_rmem = 4096 87380 16777216       # TCP receive buffer
net.ipv4.tcp_wmem = 4096 65536 16777216       # TCP send buffer
```

## 7.7 Resource Limits

```bash
# View current limits
ulimit -a                          # All limits for current shell
ulimit -n                          # Max open files
ulimit -u                          # Max processes
cat /proc/<pid>/limits             # Limits for specific process

# Set limits (current shell)
ulimit -n 65535                    # Set max open files

# Permanent limits (/etc/security/limits.conf)
# Format: <domain> <type> <item> <value>

*               soft    nofile          65535
*               hard    nofile          65535
*               soft    nproc           65535
*               hard    nproc           65535
root            soft    nofile          65535
root            hard    nofile          65535
@docker         soft    nproc           unlimited

# For systemd services, set in unit file:
[Service]
LimitNOFILE=65535
LimitNPROC=65535
LimitCORE=infinity

# Common limit items:
# nofile   : Max open file descriptors
# nproc    : Max number of processes
# memlock  : Max locked-in-memory address space
# as       : Max address space (virtual memory)
# core     : Max core file size
# data     : Max data segment size
# stack    : Max stack size
```

---

This concludes Part 2 covering advanced Linux topics including user management, permissions, systemd, packages, storage, and performance tuning.

**Continue to Part 3 for:**
- Complete Networking Deep Dive
- TCP/IP, DNS, Firewalls
- Network Troubleshooting
