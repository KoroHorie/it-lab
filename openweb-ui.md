# Open WebUI - Reset Admin Password (SQLite + bcrypt)

> Reset an Open WebUI user's password without deleting chats, models, or application data.

---

## Overview

This guide resets the password directly in the SQLite database used by Open WebUI.

**Requirements**

- Shell access to the Open WebUI container
- Python 3 installed (included with Open WebUI)
- Access to the `webui.db` database

---

# Step 1 - Locate the Database

Find the SQLite database.

```bash
find / -name "webui.db" 2>/dev/null
```

Example:

```text
/app/backend/data/webui.db
```

---

# Step 2 - Verify the Database Tables

List all tables.

```bash
python3 - <<'PY'
import sqlite3

db="/app/backend/data/webui.db"

conn=sqlite3.connect(db)
cur=conn.cursor()

cur.execute("SELECT name FROM sqlite_master WHERE type='table';")

for row in cur.fetchall():
    print(row[0])

conn.close()
PY
```

Example output:

```
auth
user
chat
config
knowledge
model
...
```

---

# Step 3 - Inspect the Authentication Table

Check the schema.

```bash
python3 - <<'PY'
import sqlite3

conn = sqlite3.connect("/app/backend/data/webui.db")
cur = conn.cursor()

cur.execute("PRAGMA table_info(auth)")

for row in cur.fetchall():
    print(row)

conn.close()
PY
```

Example:

```
id
email
password
active
```

---

# Step 4 - List Existing Accounts

View all authentication records.

```bash
python3 - <<'PY'
import sqlite3

conn = sqlite3.connect("/app/backend/data/webui.db")
cur = conn.cursor()

cur.execute("SELECT id, email, active FROM auth")

for row in cur.fetchall():
    print(row)

conn.close()
PY
```

Example:

```
('xxxxxxxx', 'admin@example.com', 1)
```

---

# Step 5 - Generate a New bcrypt Password Hash

Generate a bcrypt hash using Python.

```bash
python3 - <<'PY'
import bcrypt

password = b"TempPassword123!"
hashed = bcrypt.hashpw(password, bcrypt.gensalt())

print(hashed.decode())
PY
```

Example output:

```
$2b$12$GevAVIR0wAmtMUUBtSG.W.VXC5mDLRFcEyWlBXQ1n9lYz6IPFsnd.
```

> **Important**
>
> Copy the **entire** hash exactly as printed.
>
> The trailing `.` is part of the hash.

---

# Step 6 - Update the Password

Replace:

- `<HASH>` with the generated bcrypt hash
- `<EMAIL>` with the actual account email

```bash
python3 - <<'PY'
import sqlite3

new_hash = "<HASH>"
email = "<EMAIL>"

conn = sqlite3.connect("/app/backend/data/webui.db")
cur = conn.cursor()

cur.execute(
    "UPDATE auth SET password=? WHERE email=?",
    (new_hash, email)
)

print("Rows Updated:", cur.rowcount)

conn.commit()
conn.close()
PY
```

Expected:

```
Rows Updated: 1
```

If the result is:

```
Rows Updated: 0
```

then:

- the email address is incorrect
- no account matched the query

---

# Step 7 - Restart Open WebUI

Restart the application.

TrueNAS:

```
Apps
    → Open WebUI
        → Restart
```

or restart the Docker container.

---

# Step 8 - Login

Use:

```
Email:
<EMAIL>

Password:
TempPassword123!
```

---

# Verify bcrypt Installation

Check whether bcrypt is available.

```bash
python3 -c "import bcrypt; print('bcrypt installed')"
```

Expected:

```
bcrypt installed
```

---

# Useful Python Commands

## Show Tables

```python
cur.execute("SELECT name FROM sqlite_master WHERE type='table';")
```

---

## Show auth Schema

```python
cur.execute("PRAGMA table_info(auth)")
```

---

## Show Accounts

```python
cur.execute("SELECT id,email,active FROM auth")
```

---

## Update Password

```python
cur.execute(
    "UPDATE auth SET password=? WHERE email=?",
    (hash, email)
)
```

---

# Common Issues

## `sqlite3: command not found`

No problem.

Python already includes the SQLite module:

```python
import sqlite3
```

No need to install `sqlite3`.

---

## `apt install` Fails

Example:

```
setgroups (1: Operation not permitted)
```

Common on:

- TrueNAS Apps
- Rootless Docker
- Restricted containers

Installing packages inside the container is usually blocked.

Use Python instead.

---

## `Rows Updated: 0`

Your email in the SQL query did not match any records.

Verify the email first:

```python
SELECT id,email FROM auth;
```

---

## Password Still Doesn't Work

Verify:

- correct email
- copied the **entire** bcrypt hash
- included the trailing `.`
- restarted Open WebUI
- updated exactly one row

---

# Database Tables

Authentication credentials:

```
auth
```

Contains:

- email
- password (bcrypt)
- active

---

User information:

```
user
```

Contains:

- profile information
- user metadata

Passwords are **not** stored here.

---

# Summary

1. Locate `webui.db`
2. Inspect the `auth` table
3. Generate a bcrypt hash
4. Update the password in the `auth` table
5. Confirm `Rows Updated: 1`
6. Restart Open WebUI
7. Log in with the new password

---

**Tested Environment**

- Open WebUI
- SQLite (`webui.db`)
- Python 3
- bcrypt
- TrueNAS SCALE (Docker App)
