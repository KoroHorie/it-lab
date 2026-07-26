# Phase 3 – Users and Security
# Linux Users – Identity, User Databases, and User Creation

---

# Lesson Goal

Understand how Linux identifies users internally, where user information is stored, and why professional administrators use `useradd` instead of manually editing system databases.

---

# Linux Uses Identity, Not Usernames

Humans recognize users by name.

Example:

```
anthony
alice
bob
```

Linux recognizes users by their **User ID (UID)**.

Example:

```
1000
1001
1002
```

The username is simply a human-readable label.

Linux internally trusts the UID.

Think of it like:

```
National ID Number

↓

Never changes

↓

Name can change
```

Linux works exactly the same way.

---

# Why Linux Uses UIDs

Suppose:

```
alice

UID = 1001
```

Alice owns thousands of files.

Internally, Linux stores:

```
Owner UID = 1001
```

NOT

```
Owner = alice
```

If Alice changes her username to:

```
finance.alice
```

Linux does **not** need to change ownership of every file.

Only the username inside `/etc/passwd` changes.

The UID remains:

```
1001
```

Therefore every file still belongs to Alice.

---

# GID Works the Same Way

Groups also have identities.

Humans see:

```
developers
finance
wheel
```

Linux stores:

```
1000
1010
10
```

These are Group IDs (GIDs).

Again...

Linux trusts numbers.

Humans read names.

---

# The `id` Command

Example:

```bash
id
```

Output:

```
uid=1000(centos)
gid=1000(centos)
groups=1000(centos),10(wheel)
```

The command displays both:

- Numeric identity (UID/GID)
- Human-readable names

This makes it easy for administrators to understand ownership while Linux continues using numeric identities internally.

---

# `/etc/group`

Example entry:

```
wheel:x:10:centos
```

Fields:

```
Group Name

↓

wheel

Password Placeholder

↓

x

Group ID (GID)

↓

10

Members

↓

centos
```

---

# Primary vs Secondary Groups

Primary Group

Stored in:

```
/etc/passwd
```

Example:

```
alice:x:1001:1000
             ↑
      Primary GID
```

Secondary Groups

Stored in:

```
/etc/group
```

Example:

```
finance:x:1010:alice
wheel:x:10:alice
```

Linux separates these two intentionally.

---

# Why Primary Groups Are NOT Listed in `/etc/group`

Example:

```
developers:x:1000:
```

Notice Alice is **not** listed.

Reason:

Linux already knows Alice's primary group from:

```
/etc/passwd
```

Duplicating the information would be unnecessary.

Linux stores:

Primary Membership

→ `/etc/passwd`

Secondary Membership

→ `/etc/group`

---

# Default File Ownership

When Alice creates:

```bash
touch report.txt
```

Linux automatically assigns:

```
Owner:

alice

Group:

developers
```

The group assigned is always the user's **primary group** unless otherwise configured.

---

# Why `useradd` Exists

Technically, Linux stores users inside plain text files.

Example:

```
/etc/passwd
/etc/shadow
/etc/group
```

Therefore...

Yes.

A user could be manually inserted into `/etc/passwd`.

However, professional administrators **never** create users that way.

---

# What `useradd` Actually Does

Running:

```bash
useradd john
```

performs multiple operations automatically.

```
Create entry in /etc/passwd

↓

Create entry in /etc/shadow

↓

Create primary group

↓

Update /etc/group

↓

Create home directory

↓

Copy files from /etc/skel

↓

Assign ownership

↓

Done
```

Instead of editing multiple databases manually, Linux performs everything consistently.

---

# Default User Creation Settings

Linux stores default values here:

```
/etc/default/useradd
```

Common settings:

```
HOME=/home

SHELL=/bin/bash

SKEL=/etc/skel

INACTIVE=-1

EXPIRE=
```

These determine:

- Default home directory
- Default login shell
- Skeleton directory
- Default inactive period
- Default expiration date

---

# `/etc/skel`

Whenever Linux creates:

```
/home/john
```

It copies default files from:

```
/etc/skel
```

Examples:

```
.bashrc

.bash_profile

.bash_logout
```

This gives every new user a working shell environment immediately.

---

# Does `useradd` Set a Password?

No.

Running:

```bash
useradd john
```

creates an account but does **not** assign a usable password.

Inside `/etc/shadow` you'll typically find:

```
john:!:
```

or

```
john:!!:
```

The symbols:

```
!

or

!!
```

mean:

```
No usable password exists.
```

The account cannot authenticate until a password is assigned.

---

# Assigning a Password

Use:

```bash
passwd john
```

Linux then replaces:

```
!
```

with a real password hash.

Example:

```
$6$...
```

Now the account can authenticate successfully.

---

# Production Mindset

Creating a user is normally a two-step process.

```
useradd john

↓

Create account

↓

passwd john

↓

Assign password

↓

User can log in
```

---

# Key Takeaways

- Linux trusts UIDs and GIDs, not usernames or group names.
- Usernames are labels mapped to numeric identities.
- Primary groups are stored in `/etc/passwd`.
- Secondary groups are stored in `/etc/group`.
- New files inherit the creator's primary group.
- `useradd` automates creation across multiple system databases.
- Default user creation settings are stored in `/etc/default/useradd`.
- Default home directory contents come from `/etc/skel`.
- `useradd` does not assign passwords.
- `passwd` creates the password hash inside `/etc/shadow`.

---

# Commands Learned

```bash
id

grep "^username:" /etc/passwd

grep "^groupname:" /etc/group

cat /etc/default/useradd

useradd username

passwd username
```

---

# Production Mindset

Never manually create users by editing:

```
/etc/passwd
```

unless performing advanced system recovery or repairing a broken user database.

Always use:

```
useradd
```

because it safely updates all required Linux user databases consistently.
