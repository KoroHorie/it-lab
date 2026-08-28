# Phase 3 - User Administration
# Password Management (`passwd`) and Password Aging (`chage`)

---

# Why Linux Has `passwd`

Linux stores user password information inside:

```
/etc/shadow
```

Although this file can technically be edited manually, **administrators should never modify it directly**.

Instead, Linux provides:

```bash
passwd
```

The `passwd` command safely manages user passwords by:

- Hashing passwords correctly.
- Updating `/etc/shadow`.
- Maintaining password aging information.
- Preventing accidental corruption of the shadow database.

In production, administrators always allow Linux to manage its own authentication database.

---

# Production Scenario

HR calls.

> John forgot his password.

The account already exists.

Instead of editing `/etc/shadow`, simply reset the password.

```bash
passwd john
```

Linux prompts for a new password and automatically stores its encrypted hash.

---

# Common Production Commands

### Set or Reset Password

```bash
passwd username
```

Example:

```bash
passwd john
```

---

### Lock a User Account

```bash
passwd -l username
```

Example:

```bash
passwd -l john
```

Purpose:

Prevent password authentication without deleting the account.

Internally Linux places:

```
!
```

before the password hash inside `/etc/shadow`.

Example:

Before:

```
john:$6$HASH...
```

After:

```
john:!$6$HASH...
```

The original password hash is preserved but Linux refuses to authenticate it.

Common use cases:

- Employee resignation
- Leave of absence
- Temporary account suspension

---

### Unlock a User Account

```bash
passwd -u username
```

Example:

```bash
passwd -u john
```

Linux removes the leading `!`, allowing password authentication again.

---

### Force Password Change at Next Login

```bash
passwd -e username
```

Example:

```bash
passwd -e john
```

Commonly used after:

- Password resets
- New employee onboarding
- Security incidents

Upon next login Linux immediately requires the user to create a new password.

---

# Production Takeaways

`passwd` is the standard tool for password administration.

Common commands used by Linux administrators:

```bash
passwd username
passwd -l username
passwd -u username
passwd -e username
```

---

# Why Linux Has `chage`

Companies usually enforce password policies.

Examples:

- Password expires every 90 days.
- Warn users 7 days before expiration.
- Force password change after onboarding.

Instead of administrators manually tracking password dates, Linux records password aging information inside:

```
/etc/shadow
```

The command used to manage this information is:

```bash
chage
```

---

# Password Aging Fields

Example:

```
john:$6$HASH:21000:0:90:7:::
```

Important fields:

| Field | Meaning |
|--------|---------|
| 21000 | Last password change (days since Unix Epoch) |
| 0 | Minimum days before password can be changed again |
| 90 | Maximum password age |
| 7 | Warning period before expiration |

Linux automatically calculates expiration based on these values.

---

# Common Production Commands

### View Password Aging Information

```bash
chage -l username
```

Example:

```bash
chage -l john
```

Displays:

- Last password change
- Password expiration
- Password inactivity
- Account expiration
- Minimum password age
- Maximum password age
- Warning period

---

### Force Immediate Password Expiration

```bash
chage -d 0 username
```

Example:

```bash
chage -d 0 john
```

Forces the user to change their password during the next login.

Functionally similar to:

```bash
passwd -e john
```

---

### Set Maximum Password Age

```bash
chage -M 90 username
```

Example:

```bash
chage -M 90 john
```

Password expires after 90 days.

---

### Set Warning Period

```bash
chage -W 7 username
```

Example:

```bash
chage -W 7 john
```

Linux begins warning the user seven days before password expiration.

---

### Disable Password Expiration

```bash
chage -M 99999 username
```

Commonly used for service accounts that are not intended for interactive logins.

---

### Set Account Expiration Date

```bash
chage -E YYYY-MM-DD username
```

Example:

```bash
chage -E 2026-12-31 john
```

Purpose:

Automatically disables the account after the specified date.

Commonly used for:

- Contractors
- Vendors
- Temporary employees
- Interns

---

### Remove Account Expiration

```bash
chage -E -1 username
```

Example:

```bash
chage -E -1 john
```

Purpose:

Removes the account expiration date, allowing the account to remain active indefinitely.

---

# Understanding the Difference

Although both are managed by `chage`, **password expiration** and **account expiration** are completely different concepts.

## Password Expiration

The account remains active.

The user can still authenticate, but Linux immediately requires the password to be changed before allowing access.

Example:

- Employee password reaches 90 days.
- User enters the correct password.
- Linux prompts:

```
Your password has expired.
You must change your password now.
```

The user changes the password and continues working.

---

## Account Expiration

The account itself becomes inactive.

Linux stops the login process immediately.

The user cannot:

- Log in
- Change the password
- Authenticate using the account

Example:

A contractor is only employed until:

```
2026-12-31
```

The administrator configures:

```bash
chage -E 2026-12-31 john
```

Beginning on January 1, 2027, Linux automatically rejects all login attempts because the account has expired.

No administrator intervention is required.

---

## Production Use Cases

### Password Expiration

Common for:

- Employees
- Administrators
- Standard user accounts

Purpose:

Enforce regular password rotation according to company security policies.

---

### Account Expiration

Common for:

- Contractors
- Interns
- Vendors
- Temporary consultants

Purpose:

Automatically disable access after a known employment end date.

---

# Password Expiration vs Account Expiration

| Password Expiration | Account Expiration |
|----------------------|--------------------|
| User can still log in | Login is denied immediately |
| Linux forces password change | Linux blocks authentication |
| Account remains active | Account becomes inactive |
| Managed by `-M`, `-W`, `-d` | Managed by `-E` |

---

# Production Takeaways

`chage` manages password aging and account expiration policies.

Common commands used by Linux administrators:

```bash
chage -l username
chage -d 0 username
chage -M 90 username
chage -W 7 username
chage -E YYYY-MM-DD username
```

Instead of manually reading `/etc/shadow`, administrators usually start troubleshooting password expiration or account expiration issues with:

```bash
chage -l username
```

because it presents all password aging information in a human-readable format.

---

# Commands Learned

```bash
passwd

passwd -l

passwd -u

passwd -e

chage -l

chage -d

chage -M

chage -W

chage -E
```
