# Phase 1 — Deploying File Browser

---

# Objective

The objective of this phase is to deploy a third-party Linux application that is **not available through the operating system package manager**.

Unlike software installed using `dnf`, manually deployed applications require the Linux System Administrator to decide:

- Where the application should be downloaded.
- Where it should be installed.
- Who should own the files.
- How it will eventually be managed.

This phase focuses on following Linux filesystem standards before integrating the application with systemd.

---

# Why are we manually installing File Browser?

Most enterprise Linux servers primarily use package managers such as:

```bash
dnf
```

or

```bash
yum
```

These package managers automatically:

- Download packages
- Verify integrity
- Resolve dependencies
- Install files into standard Linux directories
- Register the software inside the RPM database

However, not every application is distributed as an RPM package.

Some software vendors distribute applications as:

- Standalone binaries
- ZIP archives
- TAR archives

File Browser is one example.

Because it is distributed as a standalone binary, the Linux administrator becomes responsible for deploying it correctly.

---

# Downloading the Application

The application was downloaded into:

```text
/tmp
```

using:

```bash
cd /tmp

wget <File Browser Download URL>
```

---

# Why did we use `/tmp`?

`/tmp` is intended for temporary files.

Linux administrators commonly use this directory to:

- Download installers
- Extract archives
- Build software
- Store temporary scripts

This keeps the rest of the filesystem clean while preparing software for deployment.

The important concept learned during this phase is:

> **Software should not remain inside `/tmp` after deployment.**

Temporary directories are not intended for production applications.

---

# Inspecting the Archive

Before extracting the application, we discussed whether it is possible to inspect the archive contents first.

Instead of immediately extracting it, Linux allows us to verify what exists inside a compressed archive.

Example:

```bash
tar -tf linux-amd64-filebrowser.tar.gz
```

This allows administrators to confirm:

- Expected filenames
- Directory structure
- Unexpected files

before extraction.

This is considered a good administration practice when working with unknown archives.

---

# Extracting File Browser

The archive was extracted using:

```bash
tar -xf linux-amd64-filebrowser.tar.gz
```

After extraction, the File Browser executable became available.

---

# Deploying the Application

Instead of executing File Browser directly from `/tmp`, the application was moved into:

```text
/opt/filebrowser
```

---

# Why `/opt`?

Linux follows the Filesystem Hierarchy Standard (FHS).

Applications installed through the operating system package manager already have predefined installation locations.

Third-party software that is manually deployed does not.

The standard location for manually installed applications is:

```text
/opt
```

This keeps manually deployed software isolated from operating system packages.

Examples include:

- Monitoring tools
- Vendor applications
- Commercial software
- Standalone binaries

File Browser falls into this category.

---

# Ownership

After deployment, the extracted binary showed:

```text
UID: 1001
GID: 1001
```

This occurred because the archive preserved the ownership information from the developer's build environment.

We changed the ownership to:

```bash
chown root:root filebrowser
```

Result:

```text
root root
```

This ensures the production server owns the deployed executable rather than preserving ownership information from another machine.

---

# Running File Browser Manually

Before integrating File Browser with systemd, we executed it manually.

```bash
./filebrowser
```

Running the application manually allowed us to verify that:

- The binary executes correctly.
- The application starts successfully.
- No immediate dependency issues exist.

---

# First Startup Behavior

During the first execution, File Browser displayed:

```text
No config file used.
```

Meaning:

No configuration file currently exists, so the application starts with its default configuration.

The application then reported:

```text
filebrowser.db can't be found.
```

File Browser automatically created:

```text
filebrowser.db
```

inside:

```text
/opt/filebrowser
```

No external database server such as:

- MariaDB
- MySQL
- PostgreSQL

was required.

Instead, File Browser uses its own embedded SQLite database.

---

# Initial Administrator Account

During first startup, File Browser automatically generated:

- Administrator username
- Random administrator password

Example:

```text
admin
Random Password
```

This allows administrators to access the web interface immediately after deployment.

---

# Listening Address

The application started successfully and listened on:

```text
127.0.0.1:8080
```

This means:

Only the local machine can currently access File Browser.

Remote clients cannot connect because the application is bound only to the loopback interface.

This behavior is suitable for future deployment behind an Nginx Reverse Proxy.

---

# What We Learned

After completing Phase 1, we learned:

- How manually installed software differs from package-managed software.
- Why Linux administrators use `/tmp` for temporary downloads.
- Why production applications belong in `/opt`.
- Why extracted archives may preserve the developer's UID and GID.
- Why ownership should be corrected after deployment.
- How to inspect archive contents before extraction.
- How File Browser performs its initial setup.
- Why no external database server was required.
- Why File Browser initially listens only on the loopback interface.

Phase 1 establishes the foundation for integrating File Browser into the operating system using **systemd**, which will be covered in the next phase.
