# Linux System Administration --- Course Roadmap

## Learning Philosophy

This course is designed as practical mentorship for becoming a
professional Linux System Administrator.

The goal is not to memorize commands. The goal is to understand **why
systems work the way they do**, investigate problems methodically, and
make safe decisions in production environments.

The core troubleshooting workflow is:

``` text
Observe
   ↓
Verify
   ↓
Investigate
   ↓
Determine Root Cause
   ↓
Plan
   ↓
Implement Solution
   ↓
Verify Solution
   ↓
Prevent Future Recurrence
```

Whenever possible, lessons use production scenarios, simulated tickets,
incidents, and troubleshooting exercises.

------------------------------------------------------------------------

# Phase 1 --- Storage & Filesystems ✅ Completed

The following topics have already been covered:

-   Storage devices
-   Partitions
-   Filesystems
-   Mounts
-   Persistent mounts
-   UUIDs
-   Hard links
-   Soft links
-   Inodes
-   LVM
-   RAID 0
-   RAID 1
-   RAID 5
-   RAID 6
-   RAID 10
-   Swap
-   Disk monitoring
-   Storage troubleshooting
-   Log rotation fundamentals

## Practical Skills Developed

-   Understanding `df` vs `du`
-   Investigating apparently missing disk space
-   Understanding deleted-but-open files
-   Understanding inode behavior
-   LVM and storage concepts
-   RAID design and trade-offs
-   Real-world disk-space forensic investigation
-   Safe investigation before making production changes

------------------------------------------------------------------------

# Phase 2 --- Services 🔵 Current Phase

## 2.1 systemd

-   What systemd is
-   PID 1
-   Service management
-   Dependencies
-   Service lifecycle
-   `systemctl`
-   Boot-time services
-   Service states

## 2.2 Units

-   Unit files
-   Unit types
-   `.service`
-   `.target`
-   `.socket`
-   `.mount`
-   `.timer`
-   Dependencies
-   `Requires=`
-   `Wants=`
-   `After=`
-   `Before=`

## 2.3 Services

-   Starting and stopping services
-   Enabling and disabling services
-   Restarting services
-   Reload vs restart
-   Service status
-   Failed services
-   Service configuration
-   Custom services

## 2.4 Targets

-   `multi-user.target`
-   `graphical.target`
-   Rescue targets
-   Emergency targets
-   Target dependencies
-   Default target
-   Boot flow

## 2.5 Linux Boot Process

The boot process will eventually be traced as:

``` text
BIOS/UEFI
   ↓
Bootloader
   ↓
Kernel
   ↓
initramfs
   ↓
systemd (PID 1)
   ↓
targets
   ↓
services
   ↓
login
```

The emphasis is on understanding **why each stage exists and how the
stages interact**.

## 2.6 Journald

-   `journalctl`
-   Persistent journals
-   Boot logs
-   Service logs
-   Filtering
-   Time-based investigation
-   Kernel messages
-   Journal retention
-   Journal storage

## 2.7 rsyslog

-   Traditional Linux logging
-   `/var/log`
-   Facilities
-   Priorities
-   Remote logging
-   Centralized logging

## 2.8 systemd Timers

-   Timer units
-   Scheduled jobs
-   Timers vs cron
-   Persistent timers
-   Calendar expressions

## 2.9 Service Troubleshooting

Major practical focus of this phase.

Example production ticket:

> **Ticket #1042:**\
> "The company's internal web application is down. Users cannot access
> it."

Investigation should follow:

``` text
Observe
   ↓
Verify
   ↓
Investigate
   ↓
Determine Root Cause
   ↓
Plan
   ↓
Implement
   ↓
Verify
   ↓
Prevent Recurrence
```

The solution should not be assumed before evidence is collected.

------------------------------------------------------------------------

# Phase 3 --- Users & Security

## Users

-   `/etc/passwd`
-   `/etc/shadow`
-   User lifecycle
-   UID/GID
-   System users

## Groups

-   Primary groups
-   Supplementary groups
-   Group administration

## PAM

-   PAM architecture
-   Authentication stack
-   Account management
-   Session management
-   PAM configuration

## Password Policies

-   Password aging
-   Password complexity
-   Account lockout policies

## File Permissions

-   Traditional permissions
-   `rwx`
-   Numeric permissions
-   Special permissions
-   SUID
-   SGID
-   Sticky bit

## ACLs

-   `getfacl`
-   `setfacl`
-   Default ACLs

## SELinux

-   Why SELinux exists
-   Enforcing, permissive, and disabled modes
-   Security contexts
-   SELinux policies
-   `restorecon`
-   `semanage`
-   Troubleshooting SELinux denials

## SSH

-   SSH architecture
-   SSH keys
-   `sshd`
-   `~/.ssh`
-   `authorized_keys`

## SSH Hardening

-   Disable root login where appropriate
-   Key-based authentication
-   Restrict users
-   Authentication policies
-   Brute-force mitigation

## sudo

-   `sudo`
-   `/etc/sudoers`
-   `/etc/sudoers.d/`
-   Least privilege
-   Delegating administrative access

------------------------------------------------------------------------

# Phase 4 --- Networking

## NetworkManager

-   NetworkManager architecture
-   Connections vs devices
-   `nmcli`

## IP Configuration

-   Static IP
-   DHCP
-   IPv4
-   IPv6

## DNS

-   DNS fundamentals
-   `/etc/resolv.conf`
-   `systemd-resolved` where applicable
-   Name resolution

## Hostname Resolution

-   `/etc/hosts`
-   NSS
-   Resolution order

## Routing

-   Routing tables
-   Default gateway
-   `ip route`
-   Multiple routes
-   Troubleshooting asymmetric routing

## firewalld

-   Zones
-   Services
-   Ports
-   Rich rules

## nftables

-   Tables
-   Chains
-   Rules
-   Stateful filtering

## iptables

-   Historical context
-   Legacy systems
-   Relationship to nftables

## Network Troubleshooting

Example incident:

> "The server can ping the gateway but cannot reach the database
> server."

Investigation should identify where connectivity fails rather than
assuming the firewall, route, DNS, or application is responsible.

------------------------------------------------------------------------

# Phase 5 --- Server Administration

## Web Servers

-   Apache
-   Nginx
-   Virtual Hosts
-   Reverse Proxy

## Databases

-   MariaDB
-   PostgreSQL

## File Services

-   Samba
-   NFS

## Scheduling

-   Cron
-   systemd timers

## Backups

-   Backup strategies
-   Full backups
-   Incremental backups
-   Differential backups
-   Backup verification
-   Restore testing
-   Retention

## Disaster Prevention

-   Failure domains
-   Single points of failure
-   Recovery planning
-   Operational documentation

------------------------------------------------------------------------

# Phase 6 --- Enterprise Linux

This phase transitions from:

> "I can administer Linux."

to:

> "I can operate Linux infrastructure professionally."

## Monitoring

-   System monitoring
-   Service monitoring
-   Disk monitoring
-   Network monitoring
-   Alerting

## Performance Monitoring

-   CPU
-   Memory
-   I/O
-   Load average
-   Process analysis
-   `top`
-   `ps`
-   `vmstat`
-   `iostat`
-   `sar`

## Log Analysis

-   Correlating logs
-   Timeline construction
-   Finding root cause
-   Incident investigation

## Automation with Ansible

-   Ansible fundamentals
-   Inventory
-   Playbooks
-   Roles
-   Idempotency
-   Configuration management

## High Availability

-   Redundancy
-   Failover
-   Clustering
-   Service availability

## Performance Tuning

-   Kernel parameters
-   Resource limits
-   I/O tuning
-   Memory behavior
-   Application/system interaction

## Capacity Planning

-   Resource trends
-   Growth forecasting
-   Bottleneck identification

## Disaster Recovery

-   RPO
-   RTO
-   Recovery procedures
-   Restore testing
-   DR exercises

## Enterprise Best Practices

-   Change management
-   Documentation
-   Least privilege
-   Monitoring
-   Backups
-   Security
-   Standardization
-   Incident response

------------------------------------------------------------------------

# Overall Progression

The course is designed to build connected understanding rather than
isolated command knowledge.

``` text
Phase 1
Storage
   ↓
"I understand what the server's data lives on."

Phase 2
Services
   ↓
"I understand what the server is actually doing."

Phase 3
Security
   ↓
"I understand who can access it and why."

Phase 4
Networking
   ↓
"I understand how the server communicates."

Phase 5
Server Administration
   ↓
"I can operate real applications and infrastructure."

Phase 6
Enterprise Administration
   ↓
"I can investigate, automate, monitor,
design, and operate production systems."
```

------------------------------------------------------------------------

# Practical Mentorship Approach

Lessons should:

1.  Explain **WHY** before **HOW**.
2.  Introduce one major concept at a time.
3.  Use realistic production scenarios.
4.  Ask questions before revealing solutions.
5.  Encourage investigation before modification.
6.  Explain what happens internally when useful.
7.  Connect new concepts to previously learned material.
8.  Increase difficulty gradually.
9.  Emphasize safe production practices.
10. Verify understanding before moving to the next topic.

When a topic is already understood at a production level, move beyond
beginner explanations into:

-   Trade-offs
-   Failure modes
-   Operational risks
-   Design decisions
-   Best practices
-   Enterprise considerations

------------------------------------------------------------------------

# Current Position

**Phase 1:** Completed ✅

**Phase 2:** Next --- `systemd`

The immediate next lesson is:

> **systemd --- Why Linux Needs a Service Manager and What PID 1
> Actually Does**

The goal is to understand systemd before relying on `systemctl` commands
mechanically.
