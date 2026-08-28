# Phase 3 — Users and Security

# User Identity Files and Linux Account Structure

---

# Objective

Understand how Linux stores user identities, authentication data, and group information.

At the end of this lesson, you should understand:

- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- UID
- GID
- Primary Groups
- Secondary Groups
- Why `useradd` exists

---

# Linux User Databases

Linux stores user information across three important files.

| File | Purpose |
|------|---------|
| `/etc/passwd` | Stores user identity information |
| `/etc/shadow` | Stores password hashes and password policies |
| `/etc/group` | Stores group definitions and secondary group memberships |

Each file has its own responsibility.

---

# /etc/passwd

## Purpose

Stores **identity information**.

It answers:

- Who is the user?
- What is their UID?
- What is their Primary Group?
- Where is their Home Directory?
- Which Login Shell should Linux start?

Example

```text
centos:x:1000:1000:centos:/home/centos:/bin/bash
```

---

## Field Breakdown

```
Username

↓

centos

Password Placeholder

↓

x

UID

↓

1000

Primary GID

↓

1000

Description (GECOS)

↓

centos

Home Directory

↓

/home/centos

Login Shell

↓

/bin/bash
```

---

# Types of Users

## Super User

```
root
```

Characteristics

- UID = 0
- Highest privilege
- Full system access

---

## Regular Users

Examples

```
centos
anthony
alice
```

Characteristics

- Human users
- Login shell
- Home directory
- Everyday administration

---

## System Users

Examples

```
sshd
nginx
daemon
tcpdump
shutdown
selinux
```

Characteristics

- Non-human accounts
- Used by services
- Usually use

```
/sbin/nologin
```

instead of `/bin/bash`.

---

# Why Passwords Are Not Stored Here

Linux used to store password hashes inside `/etc/passwd`.

Since everyone can read `/etc/passwd`, that became a huge security problem.

Modern Linux replaces the password with

```
x
```

meaning

> The actual password hash is stored in `/etc/shadow`.

---

# /etc/shadow

## Purpose

Stores:

- Password Hashes
- Password Aging
- Password Expiration
- Account Expiration

Only root can read this file.

Example

```text
centos:$6$I.aP8MjN....::0:99999:7:::
```

---

# /etc/shadow Fields

## Field 1 — Username

Example

```
centos
```

Purpose

Identifies which authentication record belongs to which user.

Linux first finds the user inside `/etc/passwd`, then retrieves the matching authentication record inside `/etc/shadow`.

---

## Field 2 — Password Hash

Example

```
$6$Salt$Hash
```

Meaning

```
Hash Algorithm

↓

$6$

↓

SHA-512

Salt

↓

Random Value

Hash

↓

Encrypted Password
```

---

### How Linux Verifies Passwords

Linux never decrypts passwords.

Instead it performs:

```
User Types Password

↓

Hash Password Again

↓

Use Same Algorithm

↓

Use Same Salt

↓

Generate New Hash

↓

Compare With Stored Hash

↓

Match?

↓

Authentication Successful
```

---

### Why Salt Exists

Without Salt

```
Password123

↓

SHA-512

↓

Same Hash
```

Attackers immediately know users share the same password.

With Salt

```
Password123

+

Random Salt

↓

Different Hash
```

Same password.

Different hashes.

This significantly increases resistance against rainbow table attacks.

---

## Field 3 — Last Password Change

Linux stores

```
Days since the Unix Epoch

↓

January 1, 1970
```

The Unix Epoch is Linux's reference starting date for counting time.

Instead of storing

```
July 25, 2026
```

Linux stores

```
20396
```

meaning

```
20,396 days have passed since January 1, 1970.
```

Linux prefers storing integers because calculations become much easier.

Example

```
Today's Day Count

↓

20420

-

Last Password Change

↓

20396

=

24 Days
```

Linux now knows:

- Password age
- Whether it expired
- Whether warning messages should appear
- Whether account policies should be enforced

This field works together with:

- Minimum Password Age
- Maximum Password Age
- Warning Days
- Inactive Days

---

## Field 4 — Minimum Password Age

Example

```
0
```

Meaning

Minimum number of days before the password can be changed again.

Example

```
0
```

User can immediately change their password.

Production environments may require several days to prevent password cycling.

---

## Field 5 — Maximum Password Age

Example

```
99999
```

Meaning

Maximum number of days before the password expires.

Example

```
99999
```

Practically means

```
Never Expires
```

Enterprise environments commonly configure values such as

```
90
```

forcing password rotation.

---

## Field 6 — Warning Days

Example

```
7
```

Meaning

Linux begins warning users before password expiration.

Example

```
Your password will expire in 7 days.
```

---

## Field 7 — Inactive Days

Example

```
30
```

Meaning

After the password expires...

Linux waits another 30 days before disabling the account.

```
Password Expired

↓

30-Day Grace Period

↓

Account Disabled
```

---

## Field 8 — Account Expiration

Purpose

Disables the account itself regardless of the password.

Common production use cases

- Contractors
- Temporary Employees
- Interns
- Project Accounts

Once the expiration date is reached...

Linux refuses authentication.

---

## Field 9 — Reserved

Currently unused.

Reserved for future Linux implementation.

---

# Identity vs Authentication

One of the most important concepts.

## /etc/passwd

Stores

```
Identity
```

Contains

- Username
- UID
- Primary GID
- Home Directory
- Login Shell

---

## /etc/shadow

Stores

```
Authentication
```

Contains

- Password Hash
- Password Aging
- Account Expiration

---

# High-Level Authentication Flow

At this stage we only understand the high-level flow.

```
Username Entered

↓

/etc/passwd

↓

User Exists?

↓

/etc/shadow

↓

Password Verified?

↓

Authentication Successful

↓

Login Shell Starts

↓

User Session
```

The internal authentication engine (PAM) will be covered later.

---

# UID

Linux does **not** identify users by username.

Humans think

```
Anthony
Alice
Bob
```

Linux thinks

```
1000
1001
1002
```

These are UIDs.

The username is merely a label.

Internally Linux stores ownership using the UID.

Instead of

```
Owner = Alice
```

Linux stores

```
Owner UID = 1001
```

When running

```bash
ls -l
```

Linux translates

```
1001

↓

alice
```

using `/etc/passwd`.

---

# GID

Groups work exactly the same.

Humans think

```
developers
finance
wheel
```

Linux thinks

```
1000
1010
10
```

Again...

Names are labels.

Numbers are identities.

---

# /etc/group

Purpose

Stores

- Group Names
- GIDs
- Secondary Group Membership

Example

```text
wheel:x:10:centos
```

Field Breakdown

```
Group Name

↓

wheel

Password Placeholder

↓

x

GID

↓

10

Members

↓

centos
```

---

# Primary Group

Every user has exactly one Primary Group.

Stored inside

```
/etc/passwd
```

Example

```text
alice:x:1001:1000
```

Primary GID

```
1000
```

---

# Secondary Groups

Stored inside

```
/etc/group
```

Example

```text
finance:x:1010:alice
wheel:x:10:alice
```

Meaning

Alice also belongs to:

- finance
- wheel

---

# Why Primary Groups Are Not Listed Inside /etc/group

Linux already knows the Primary Group from

```
/etc/passwd
```

Repeating it inside `/etc/group` would duplicate information.

Linux stores

```
Primary Group

↓

/etc/passwd

Secondary Groups

↓

/etc/group
```

---

# New File Ownership

Suppose Alice creates

```bash
touch report.txt
```

Linux automatically assigns

```
Owner

↓

Alice

Group

↓

Primary Group
```

Not one of the secondary groups.

---

# Can We Create Users By Editing /etc/passwd?

Technically...

Yes.

Linux immediately recognizes new entries because `/etc/passwd` is a plain text database.

However...

Only creating `/etc/passwd` does **NOT**:

- Create `/etc/shadow`
- Create Home Directory
- Create Primary Group
- Update `/etc/group`
- Copy `/etc/skel`
- Assign Ownership
- Assign Permissions

---

# Why useradd Exists

`useradd` automates the entire process.

Instead of only modifying

```
/etc/passwd
```

it performs

```
Create /etc/passwd Entry

↓

Create /etc/shadow Entry

↓

Create Primary Group

↓

Update /etc/group

↓

Create Home Directory

↓

Copy /etc/skel

↓

Assign Ownership

↓

Assign Permissions
```

---

# Missing /etc/shadow Entry

Suppose you manually add

```text
anthony:x:1002:1002:Anthony:/home/anthony:/bin/bash
```

to `/etc/passwd`

but forget `/etc/shadow`.

Linux can

✅ Find the user.

But cannot

❌ Authenticate the user.

Why?

Because there is no stored password hash to compare against.

Authentication fails.

---

# Key Takeaways

- `/etc/passwd` stores **identity**.
- `/etc/shadow` stores **authentication**.
- `/etc/group` stores **group definitions**.
- Linux trusts **UIDs** and **GIDs**, not usernames and group names.
- Every user has exactly one Primary Group.
- Secondary Groups are stored in `/etc/group`.
- New files inherit the creator's Primary Group.
- Linux reads these files directly.
- `useradd` exists because creating a Linux user involves much more than writing a single line into `/etc/passwd`.
