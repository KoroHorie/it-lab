# Phase 2 — Logging (Part 2: journald vs Application Logs)

## Two Different Types of Logs

One of the most important concepts in Linux logging is understanding that **not all logs are the same**.

A Linux service such as **Nginx** actually produces **two different categories of logs**.

---

# Category 1 — Application Logs

Application logs are logs that the application **intentionally writes itself**.

The application decides:

- Where to store them.
- What information to record.
- The format of the log entries.

Examples:

```
/var/log/nginx/access.log
/var/log/nginx/error.log

/var/log/kibana/kibana.log

/var/log/elasticsearch/elasticsearch.log
```

These logs describe the application's internal behavior.

Example:

```
Client requested GET /
Returned HTTP 200

PHP upstream timed out

Database connection failed
```

Application logs answer questions such as:

- What requests did the application process?
- What runtime errors occurred?
- What internal operations failed?

These logs belong to the application itself.

---

# Category 2 — Service Logs (journald)

When systemd starts a service, it becomes the parent process of that service.

Example:

```
systemd (PID 1)
        │
        ▼
    nginx.service
```

Every Linux process has standard streams:

```
stdin
stdout
stderr
```

Whenever the service prints messages to:

```
stdout
stderr
```

systemd captures those outputs and forwards them to:

```
systemd-journald
```

These become service logs.

Examples:

```
Service started

Service stopped

Configuration test failed

Permission denied

Failed to bind to port 80

Process exited with status code 1
```

Unlike application logs, these messages describe **the lifecycle of the service**.

---

# Relationship Between systemd and journald

systemd launches services.

journald records what those services print to stdout and stderr.

Conceptually:

```
systemd
     │
     ▼
Launches nginx.service
     │
     ▼
Captures stdout / stderr
     │
     ▼
systemd-journald
     │
     ▼
journalctl
```

This explains why `systemctl status` and `journalctl` often show similar messages.

---

# Why systemctl status Shows Log Messages

When executing:

```bash
systemctl status nginx.service
```

systemctl does **not** store log messages.

Instead, it queries journald and displays the most recent entries.

Example:

```
Active: failed

...

Jul 25 13:05:18 nginx:
Configuration test failed.
```

Those lines are retrieved from journald.

---

# Where Does journald Store Logs?

Unlike traditional text log files, journald stores logs in a **binary journal database**.

Temporary logs:

```
/run/log/journal/
```

Persistent logs (if enabled):

```
/var/log/journal/
```

Because the journal is binary, it cannot be read using:

```bash
cat
tail
less
```

Instead, administrators use:

```bash
journalctl
```

---

# Investigation Workflow

When troubleshooting a failed service, a Linux administrator usually follows this order:

```
User reports issue
        │
        ▼
systemctl status
        │
        ▼
journalctl
        │
        ▼
Application Logs
        │
        ▼
Configuration
        │
        ▼
Implement Fix
```

---

# Why This Order?

## Step 1 — systemctl status

Purpose:

Determine the current health of the service.

Questions answered:

- Is it running?
- Did it fail?
- Exit code?
- Recent log messages?

---

## Step 2 — journalctl

Purpose:

Understand how the service behaved.

Questions answered:

- Why did systemd consider it failed?
- What messages were printed during startup?
- What happened immediately before failure?

---

## Step 3 — Application Logs

Purpose:

Understand the application's internal behavior.

Questions answered:

- What runtime error occurred?
- What request caused the issue?
- What subsystem failed?

---

# Example Investigation

Users report:

```
"Kibana is unavailable."
```

Administrator workflow:

```bash
systemctl status kibana
```

↓

```bash
journalctl -u kibana
```

↓

```bash
tail -f /var/log/kibana/kibana.log
```

Each step narrows the investigation.

---

# Conceptual Architecture

```
                 User Request
                      │
                      ▼
                   Application
                 (Example: nginx)
                ┌───────────────┐
                │               │
                ▼               ▼
      Application Logs     stdout / stderr
      (/var/log/...)            │
                                ▼
                           systemd
                                │
                                ▼
                       systemd-journald
                                │
                                ▼
                           journalctl
```

---

# Key Takeaways

## 1. Linux has multiple logging sources.

Not every log comes from journald.

Applications often maintain their own log files.

---

## 2. journald records service lifecycle events.

Examples:

- Service started
- Service stopped
- Startup failures
- stdout
- stderr

---

## 3. Application logs record application-specific activity.

Examples:

- HTTP requests
- Runtime errors
- Internal failures
- Business logic events

---

## 4. systemctl status retrieves recent log messages from journald.

It does not store the logs itself.

---

## 5. Professional Investigation Order

```
systemctl status
        ↓
journalctl
        ↓
Application Logs
        ↓
Configuration
```

This minimizes guessing and helps narrow the investigation efficiently.
