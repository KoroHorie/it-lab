# Part 4 — rsyslog

## Overview

After understanding `systemd-journald`, the next component is **rsyslog**.

Unlike journald, which is responsible for collecting logs into the system journal, **rsyslog is a log processing and routing service**.

Its primary responsibilities are:

- Organizing logs into traditional text files under `/var/log`
- Applying rules to determine where logs should go
- Optionally forwarding logs to remote log collectors or SIEM platforms

---

# Logging Flow

```
Applications / Kernel
        │
        ▼
systemd-journald
        │
        ▼
      rsyslog
       │
       ├──────────────► Internal Storage (/var/log/*)
       │
       └──────────────► External Log Collectors
                        (SIEM, ELK, Splunk, Syslog Server)
```

This is the standard logging pipeline on many Enterprise Linux systems.

---

# Relationship between journald and rsyslog

These two services are **not competitors**.

Instead, they work together.

### journald

Responsible for:

- Receiving log messages from the kernel
- Receiving log messages from services
- Receiving log messages from applications
- Storing them inside the system journal

Think of journald as the **central collector**.

---

### rsyslog

Responsible for:

- Reading logs from journald
- Applying routing rules
- Writing logs into traditional text log files
- Optionally forwarding logs to another server

Think of rsyslog as the **traffic controller**.

---

# Internal Logging

On a normal Linux server, rsyslog usually organizes logs into different files.

Examples:

Authentication logs

```
/var/log/secure
```

General system logs

```
/var/log/messages
```

Cron logs

```
/var/log/cron
```

Mail logs

```
/var/log/maillog
```

Boot logs

```
/var/log/boot.log
```

Instead of storing every event in one giant file, rsyslog separates them according to predefined rules.

---

# External Logging

Besides writing logs locally, rsyslog can also send logs to another machine.

Example architecture:

```
Linux Server
      │
      ▼
systemd-journald
      │
      ▼
rsyslog
      │
      ▼
SIEM
```

Possible centralized logging platforms:

- ELK Stack
- Splunk
- Graylog
- QRadar
- Dedicated Syslog Server

This allows hundreds or thousands of Linux servers to send their logs to a single location for monitoring and incident response.

---

# Where is rsyslog configured?

Main configuration:

```
/etc/rsyslog.conf
```

Additional modular configurations:

```
/etc/rsyslog.d/
```

Enterprise environments usually place custom rules inside:

```
/etc/rsyslog.d/
```

instead of modifying the main configuration directly.

---

# Understanding rsyslog Rules

The configuration is simply a collection of routing rules.

Example:

```conf
authpriv.* action(type="omfile" file="/var/log/secure")
```

Meaning:

```
IF

Authentication log

↓

THEN

Write into

↓

/var/log/secure
```

Another example:

```conf
cron.* action(type="omfile" file="/var/log/cron")
```

Meaning:

```
IF

Cron log

↓

THEN

Write into

↓

/var/log/cron
```

Another example:

```conf
mail.* action(type="omfile" file="/var/log/maillog")
```

Meaning:

```
IF

Mail log

↓

THEN

Write into

↓

/var/log/maillog
```

---

# Conceptual Model

Think of rsyslog as a giant decision engine.

```
Log Message

↓

Match Rule?

↓

Yes

↓

Perform Action

↓

Destination
```

The destination could be:

- A local log file
- Another Linux server
- A SIEM
- ELK
- Splunk

---

# Internal vs External Logging

Internal Logging

Purpose:

Store logs locally for Linux administrators.

Examples:

```
/var/log/messages
/var/log/secure
/var/log/cron
```

---

External Logging

Purpose:

Centralize logs from many servers.

Example:

```
Linux Server

↓

rsyslog

↓

SIEM
```

Both can happen simultaneously.

A log can be:

- Stored locally
- Forwarded remotely

at the same time.

---

# Why Administrators Care About rsyslog

A Linux System Administrator typically interacts with rsyslog when:

- Configuring centralized logging
- Troubleshooting missing log files
- Investigating why logs are not reaching a SIEM
- Modifying log routing policies

Most day-to-day troubleshooting is performed using:

- `journalctl`
- `/var/log/*`

rather than directly editing rsyslog rules.

---

# Key Takeaways

- `systemd-journald` collects logs.
- `rsyslog` processes and routes logs.
- rsyslog organizes logs into traditional files under `/var/log`.
- rsyslog can also forward logs to centralized logging systems.
- rsyslog follows routing rules defined in its configuration.
- The primary configuration files are:
  - `/etc/rsyslog.conf`
  - `/etc/rsyslog.d/`
- Learning the concepts is more important than memorizing rsyslog syntax. Configuration syntax is typically learned when a production task requires it.
