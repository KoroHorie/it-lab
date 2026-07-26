# Linux System Administration Mentorship
# Phase 3 - Users and Security
# Part 1 - Linux Users, Identities, and Groups

---

# Objective

Understand what a Linux user really is internally before learning commands such as useradd, usermod, groupadd, passwd, sudo, ACLs, SSH, and SELinux.

Production mindset:

Never think of a Linux user as simply "someone who logs in."

Linux sees every user as an **identity**.

---

# Identity Before Commands

A Junior System Administrator usually thinks:

"A user is a person."

Linux does NOT.

Linux thinks:

User = Identity

Every identity has:

- UID (User Identifier)
- Primary GID (Primary Group Identifier)
- Optional Supplementary Groups
- Home Directory
- Login Shell
- Password
- Account Properties

Everything else is simply metadata attached to that identity.

---

# Types of Linux Users

There are three common categories.

---

## 1. Regular Users

Purpose:

Human accounts used for everyday work.

Examples:

- alice
- bob
- anthony

Characteristics:

- Can login
- Has home directory
- Uses a login shell
- Owns files
- Runs processes

Example:

alice

↓

UID 1001

↓

Primary Group developers

↓

Home

/home/alice

↓

Shell

/bin/bash

---

## 2. System Users

Purpose:

Service identities.

Examples:

- nginx
- mysql
- postgres
- chrony
- sshd

Characteristics:

- Usually cannot login
- Often no home directory
- Own files
- Own processes
- Run services

Production example:

nginx.service

↓

Runs as nginx user

↓

Reads

/var/www/html

↓

Writes

/var/log/nginx

If permissions are incorrect, the service fails.

---

## 3. Super User (root)

Purpose:

Administrative account.

Identity:

UID = 0

Characteristics:

- Full administrative privileges
- Can manage users
- Can manage filesystems
- Can manage services
- Bypasses normal UNIX file permission checks (DAC)

Root is NOT magical.

Internally Linux recognizes:

UID = 0

---

# The Most Important Concept

Linux does NOT care about usernames.

Linux cares about UIDs.

Humans see:

alice

Linux sees:

1001

---

Example:

File ownership

Internally:

Owner UID = 1001

When running:

ls -l

Linux performs:

Owner UID

↓

Lookup

/etc/passwd

↓

1001

↓

alice

↓

Display "alice"

The username is simply a readable mapping.

The UID is the true identity.

---

# Connection to Phase 1

Remember:

Files

↓

Inodes

↓

Metadata

One inode stores:

Owner UID

Owner GID

NOT usernames.

---

# Processes Also Use Identities

Example:

Nginx

Internally:

Runs as UID 997

NOT

Runs as "nginx"

Again:

Identity

↓

UID

---

# Definition

A Linux User is:

"An operating system identity represented internally by a unique UID."

Everything else is metadata attached to that identity.

---

# Why Groups Exist

Question:

If users already have identities...

Why invent Groups?

Answer:

Permission management.

Imagine:

100 Finance employees.

Without Groups:

Grant permissions individually

100 times.

With Groups:

Finance Group

↓

Everyone inside the group automatically receives access.

Groups simplify administration.

---

# Group Identity

Groups also have identities.

GID

(Group Identifier)

Linux has:

User Identity

↓

UID

Group Identity

↓

GID

---

# Primary Group

Every user has exactly ONE primary group.

Purpose:

Default group ownership of newly created files.

Example:

alice

Primary Group

developers

Alice creates:

touch report.txt

Ownership becomes:

Owner

alice

Group

developers

Linux automatically assigns the primary group.

---

# Supplementary (Secondary) Groups

Purpose:

Grant access to additional shared resources.

Example:

alice

Primary

developers

Secondary

finance

docker

git

Alice creates:

notes.txt

Ownership:

Owner

alice

Group

developers

NOT

finance

NOT

docker

Secondary groups DO NOT become the default group ownership.

They only provide additional access rights.

---

# Company Analogy

Primary Group

↓

Official Department

Secondary Groups

↓

Additional teams you collaborate with

Example:

Primary

IT

Secondary

Security

Audit

Cloud

Official department remains IT.

---

# File Permission Evaluation

Linux checks permissions in a strict order.

Step 1

Owner?

↓

YES

↓

Use Owner permissions

↓

STOP

Otherwise

↓

Step 2

Group?

↓

YES

↓

Use Group permissions

↓

STOP

Otherwise

↓

Step 3

Others

Linux NEVER combines Owner, Group and Others permissions.

Only ONE permission class is used.

---

# Owner Example

File:

Owner

bob

Group

developers

Permissions

-rw-rw----

Alice

Primary Group

developers

Owner?

No

↓

Group?

Yes

↓

Apply Group permissions

↓

Read

Write

Access Granted

---

# Important Rule

Primary vs Secondary Groups

For permission checking

Linux DOES NOT care.

If the user belongs to the owning group...

Either as:

Primary

OR

Secondary

Permission is granted.

---

# Others

Others means:

"Everyone who is NOT the owner and NOT in the owning group."

Example:

-rw-r--rw-

Owner

alice

Group

developers

Others

Read

Write

Bob belongs only to Finance.

Owner?

No

↓

Group?

No

↓

Others?

rw-

↓

Bob can read and write.

---

# Production Rule

Confidential files almost always end with:

Others

---

Meaning:

If you are not the owner

AND

Not inside the correct group

↓

No access.

---

# chmod 777

Permissions:

rwxrwxrwx

Meaning:

Owner

Full

Group

Full

Others

Full

This is almost never appropriate on production servers.

Typical Senior SysAdmin mindset:

Never solve permission problems with chmod 777.

Investigate:

1. Owner
2. Group
3. User memberships
4. Permission bits
5. ACLs
6. SELinux

---

# Root and Permission Checks

Root

↓

UID = 0

Root is NOT treated as:

Owner

OR

Group Member

Instead,

The kernel recognizes UID 0 and bypasses traditional UNIX permission checks (DAC).

Root does NOT magically become the owner.

It simply has privileged access.

Example:

chmod 000 payroll.txt

Nobody can read it.

Except root.

---

# Important Note

Root bypasses traditional UNIX permissions.

However,

Root does NOT necessarily bypass:

- SELinux
- Immutable attributes (chattr +i)
- Read-only filesystems
- Kernel lockdown features

Modern Linux has additional security layers beyond rwx permissions.

---

# Key Takeaways

- Linux users are identities.
- UID is the true identity.
- Usernames are human-readable mappings.
- Groups have identities too (GIDs).
- Primary Group determines default group ownership.
- Secondary Groups provide additional shared access.
- Linux permission order:
  Owner → Group → Others
- Linux never combines permission classes.
- Owner status overrides group membership.
- Group membership can be either Primary or Secondary.
- Others represents everyone else.
- chmod 777 is rarely appropriate.
- Root bypasses traditional UNIX permissions through UID 0.
