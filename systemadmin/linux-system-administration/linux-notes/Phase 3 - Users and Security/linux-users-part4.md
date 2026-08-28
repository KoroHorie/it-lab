# Phase 3 — Users and Security
# User Modification (usermod) and the wheel Group

---

# Why do we need `usermod`?

Production Scenario

HR sends you a ticket:

> "Alice got promoted to DevOps."

She already has:

- Files
- Password
- SSH Keys
- UID
- Home Directory

Question:

Should we delete Alice and create another account?

**No.**

Why?

Because Linux identifies users using **UID**, not usernames.

Deleting and recreating Alice means:

- New UID
- Broken file ownership
- New password
- Possible application permission issues

Instead of recreating the account, Linux provides:

```bash
usermod
```

Its purpose is simply:

> Modify an existing user safely without replacing its identity.

---

# The Three Most Important Options

## `-g`

Changes the **Primary Group**.

Example:

```bash
usermod -g devops john
```

Before:

```
Primary Group

john
```

After:

```
Primary Group

devops
```

Only the primary group changes.

---

## `-G`

Changes **all Secondary Groups**.

This does NOT append.

It REPLACES.

Example:

Alice originally belongs to:

```
finance
wheel
developers
```

Running:

```bash
usermod -G docker alice
```

Results in:

```
docker
```

Only Docker remains.

Everything else is removed.

Primary group remains unchanged.

This is one of the easiest mistakes juniors make.

---

## `-aG`

This means:

Append (`-a`)

+

Secondary Groups (`-G`)

Example:

```bash
usermod -aG docker alice
```

Before:

```
finance
wheel
```

After:

```
finance
wheel
docker
```

Nothing is replaced.

Only appended.

---

# Hands-on Lab

Created user:

```bash
useradd john
passwd john
```

Created group:

```bash
groupadd devops
```

Changed John's primary group:

```bash
usermod -g devops john
```

---

# Observation #1

Immediately after changing John's primary group, I noticed something.

Before:

```
Owner

john

Group

john
```

After:

```
Owner

john

Group

devops
```

The home directory now reflects the new primary group.

This was something I observed during the lab.

---

# Hands-on Lab

While logged in as John:

```bash
mkdir -p devops/bash-scripts

touch script01.sh

nano script01.sh
```

Contents:

```sh
#!/bin/sh

echo "This is a test script."

name=$USER

echo "Logging in..."

sleep 0.5

echo "Hello, my name is $name"
```

Understanding:

`$USER`

↓

Represents the currently logged-in user executing the script.

Therefore:

John executes

↓

```
Hello, my name is john
```

Alice executes

↓

```
Hello, my name is alice
```

The script changes dynamically depending on who runs it.

---

# File Permissions

Configured permissions:

```bash
chmod ug+x script01.sh

chmod o-rx script01.sh
```

My goal was:

Owner

↓

Can execute

Group

↓

Can execute

Others

↓

Cannot access

---

# Hands-on Lab

Created another user:

```bash
useradd alice -p ecila
```

Then checked:

```
/etc/shadow
```

Observation:

Linux stored:

```
ecila
```

It was NOT hashed.

I then fixed it by running:

```bash
passwd alice
```

---

# Discovery

I discovered that:

```bash
useradd -p
```

expects an already encrypted password hash.

It does NOT hash plain text passwords.

Correct production workflow:

```bash
useradd alice

passwd alice
```

---

# Hands-on Lab

Added Alice into DevOps:

```bash
usermod -aG devops alice
```

Expectation:

Alice should execute John's script.

Reality:

```
Permission denied
```

---

# Observation #2

The problem was NOT the script.

The problem was:

```
/home/john
```

Alice couldn't even enter John's home directory.

Linux first traverses:

```
/home

↓

john

↓

devops

↓

bash-scripts

↓

script01.sh
```

Since Alice cannot enter:

```
/home/john
```

Linux never reaches the script.

---

# Solution

Instead of storing shared resources inside:

```
/home/john
```

Move them somewhere neutral.

Example:

```
/opt

/srv

/shared
```

Now both Alice and John can execute the script because both belong to:

```
devops
```

---

# The wheel Group

Question during the lesson:

"What exactly is wheel?"

Answer:

The wheel group allows users to use:

```bash
sudo
```

It does NOT automatically make the user root.

Example:

Without wheel:

```bash
dnf install nginx
```

↓

Permission denied.

With wheel:

```bash
sudo dnf install nginx
```

↓

Linux checks:

```
Is user allowed to use sudo?

↓

Is user in wheel?

↓

YES

↓

Ask password

↓

Run command as root
```

Notice:

The user remains a regular user.

Only that command executes with root privileges.

---

# Becoming Root Temporarily

Commands such as:

```bash
sudo -i
```

or

```bash
sudo su -
```

start a root shell.

Example:

Before:

```
john@server$
```

After:

```
root@server#
```

This is possible because the user has permission to use sudo, commonly granted through membership in the wheel group.

---

# Commands Learned

```bash
groupadd

usermod -g

usermod -G

usermod -aG

sudo

sudo -i

sudo su -
```

---

# Production Takeaways

- Never delete and recreate a user simply because their role changes.
- Modify existing accounts using `usermod`.
- `-g` changes the Primary Group.
- `-G` replaces all Secondary Groups.
- `-aG` appends Secondary Groups.
- Shared project directories should not live inside another user's home directory.
- `useradd -p` expects a hashed password.
- The wheel group allows users to use `sudo`.
- Membership in wheel does NOT permanently make someone root.
