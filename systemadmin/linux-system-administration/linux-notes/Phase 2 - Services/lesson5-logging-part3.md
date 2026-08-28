# Phase 2 — Services
# Logging (journald & rsyslog)
## Part 3 — Investigating Logs with journalctl (Production Mindset)

---

# After Understanding the Journal...

In Part 2, we learned:

- journald collects logs from the operating system and services.
- journalctl is simply the tool used to read the journal.
- Application logs still exist independently (such as Nginx's access.log and error.log).

At this point, we shift our mindset.

We are no longer asking:

> "What is journald?"

Instead, we ask:

> **"How do administrators actually investigate systems using journald?"**

This is where `journalctl` becomes a troubleshooting tool instead of just a log viewer.

---

# The First Investigation Filter — Units

The very first filter every Linux administrator learns is:

```bash
journalctl -u <unit>
```

Example:

```bash
journalctl -u nginx.service
```

Notice something important.

We are **not** asking:

> Show me every log in the operating system.

We are asking:

> Show me every journal entry that belongs to **this specific unit**.

Remember from our previous lessons:

- nginx.service is a systemd unit.
- sshd.service is a systemd unit.
- NetworkManager.service is a systemd unit.
- chronyd.service is a systemd unit.

Therefore:

```bash
journalctl -u nginx.service
```

means:

> Show me everything journald knows about nginx.service.

---

# Why Not Simply Run journalctl?

Suppose your server has been running for weeks.

During those weeks, the journal has collected logs from:

- Kernel
- NetworkManager
- systemd
- SSH
- Nginx
- Docker
- Cron
- Firewalld
- SELinux
- Hundreds of other services

Running:

```bash
journalctl
```

may easily produce tens or even hundreds of thousands of log entries.

Imagine trying to troubleshoot nginx by reading logs from Bluetooth, cron jobs, kernel PCI devices, and NetworkManager.

Impossible.

Filtering by unit immediately narrows the investigation.

---

# Example

Instead of:

```bash
journalctl
```

use:

```bash
journalctl -u nginx.service
```

Output may resemble:

```
Starting nginx...

Configuration test successful.

Started nginx.
```

Immediately, you've reduced thousands of log lines into only the lifecycle of the nginx service.

---

# Very Important

`journalctl -u nginx.service`

does **NOT** show nginx runtime errors.

It shows the **service lifecycle**.

Meaning:

- Did it start?
- Did it stop?
- Was configuration testing successful?
- Did systemd detect failure?
- Did the service exit unexpectedly?

---

# Comparing journald with Application Logs

One of the most important realizations during this lesson was understanding that journald and application logs answer different questions.

## journald

Answers:

- Did the service start?
- Did systemd detect failure?
- Was configuration testing successful?
- Was the service restarted?
- Was it terminated?

Example:

```
Started nginx.
```

```
Configuration test successful.
```

---

## access.log

Answers:

- Who visited?
- What URL did they request?
- What HTTP status code was returned?

Example:

```
GET /
200
```

Meaning:

A client successfully accessed the homepage.

---

## error.log

Answers:

- What runtime problems occurred while serving requests?

Example:

```
open()

favicon.ico

No such file or directory
```

Meaning:

The browser requested:

```
/favicon.ico
```

Nginx attempted to locate it.

The file didn't exist.

---

# The Important Lesson

When you observed:

```
favicon.ico

No such file or directory
```

you didn't panic.

Instead you reasoned:

> This custom website simply doesn't have a favicon.

Therefore:

This is **not** a serious production incident.

---

# Administrators Don't Read Logs.

They Evaluate Logs.

This is probably the biggest lesson of this section.

Many beginners think:

```
ERROR
```

means:

> The server is broken.

Professional administrators instead ask:

> Is this expected?

---

Example

Not Critical:

- favicon missing
- browser requesting nonexistent file
- client disconnected
- user closed browser tab

Potentially Critical:

- Out of memory
- Permission denied
- Segmentation fault
- Cannot bind port
- No space left on device
- DHCP timeout

Reading logs therefore requires judgement.

Not every error deserves action.

---

# journalctl -x

While investigating, we also discovered:

```bash
journalctl -x
```

This option does **not** show more logs.

Instead, it augments certain system-generated log messages with explanations from the systemd message catalog.

Think of it like documentation attached directly to certain journal entries.

Without `-x`:

```
Unit nginx.service entered failed state.
```

With `-x`:

Additional explanation appears describing why systemd generated that message.

This is useful whenever a system-generated message is unfamiliar.

It is **not** something administrators use constantly, but it can provide valuable context during investigation.

---

# Investigating Boot Sessions

We then discussed another production problem.

Imagine:

```
03:00 AM

Server crashed.

Someone rebooted it before you arrived.
```

Now the journal contains logs from multiple boot sessions.

You don't want every boot.

You only want the one relevant to your investigation.

---

## Current Boot

```bash
journalctl -b
```

Meaning:

> Show logs only from the current boot session.

Every reboot creates a new journal timeline.

Think of every boot as its own chapter.

```
Boot #1

↓

Boot #2

↓

Boot #3 (Current)
```

`journalctl -b`

shows only the current chapter.

---

## Previous Boot

Production troubleshooting becomes even more interesting.

Suppose the crash happened yesterday.

You can inspect:

```bash
journalctl -b -1
```

Meaning:

Show me the previous boot.

Likewise:

```bash
journalctl -b -2
```

shows two boots ago.

This allows administrators to investigate failures that occurred **before** a reboot.

---

# Filtering by Priority

We then introduced another very useful filter.

```bash
journalctl -p
```

Unlike `-u` (which filters by unit),

`-p` filters by **priority**.

---

# Connection with Syslog Severity

This immediately connected with prior knowledge.

Traditional Syslog priorities:

| Level | Name | Description |
|-------|------|-------------|
| 0 | Emergency | System unusable |
| 1 | Alert | Immediate action required |
| 2 | Critical | Critical condition |
| 3 | Error | Error condition |
| 4 | Warning | Warning condition |
| 5 | Notice | Significant normal condition |
| 6 | Info | Informational |
| 7 | Debug | Debugging |

Because you already knew Syslog severity levels, understanding `journalctl -p` became much easier.

---

# Examples

Errors only:

```bash
journalctl -p err
```

Warnings and anything more severe:

```bash
journalctl -p warning
```

Critical only:

```bash
journalctl -p crit
```

Debug messages:

```bash
journalctl -p debug
```

---

# Why This Matters

Imagine a journal containing:

```
250,000 log entries
```

Do you care about:

- User logged in
- Timer started
- Bluetooth initialized
- USB connected

Probably not.

If you're investigating failures:

```
journalctl -p err
```

removes enormous amounts of noise.

---

# Combining Filters

The real power comes from combining filters.

Examples:

Current boot:

```bash
journalctl -b
```

Current boot + errors:

```bash
journalctl -b -p err
```

Specific service:

```bash
journalctl -u nginx.service
```

Specific service + explanations:

```bash
journalctl -xu nginx.service
```

Specific service + current boot:

```bash
journalctl -b -u nginx.service
```

You are continuously reducing noise until only the evidence remains.

---

# NetworkManager Investigation

Instead of asking what NetworkManager's service name was, you discovered it yourself.

You searched:

```bash
systemctl list-units | grep -i network
```

which revealed:

```
NetworkManager.service
```

Then investigated:

```bash
journalctl -u NetworkManager
```

---

# What You Found

While scanning the logs, you immediately noticed one important message:

```
dhcp4 (ens160):

state changed

new lease

address=10.13.21.65
```

From that single message you concluded:

> The machine successfully obtained an IPv4 address from the DHCP server.

You did **not** need to read every log line.

That is exactly how experienced administrators investigate systems.

---

# Reading Logs Efficiently

One of the biggest lessons from this section:

Professionals do **not** read logs like novels.

Instead they:

1. Know the question they want answered.
2. Search for important events.
3. Read surrounding context only when necessary.

Example questions:

## Did DHCP succeed?

Look for:

- DHCP
- Lease
- Address

Question answered.

---

## Did nginx receive requests?

Look at:

```
access.log
```

---

## Did nginx fail to start?

Look at:

```
journalctl -u nginx.service
```

---

## Did the operating system boot correctly?

Look at:

```
journalctl -b
```

---

# Production Mindset

Notice how every journalctl option answers a different question.

`-u`

> Which service?

`-b`

> Which boot?

`-p`

> Which severity?

`-x`

> Can the system explain this message?

Instead of memorizing switches, experienced administrators ask:

> What information am I trying to obtain?

Then choose the filter that answers that question.

---

# Key Takeaways

- `journalctl -u` filters logs for a specific systemd unit.
- Journald describes the operating system and service lifecycle.
- Application logs describe runtime behaviour inside the application.
- Not every error indicates a production incident.
- Administrators evaluate whether a log entry is expected before reacting.
- `journalctl -x` augments catalog-aware messages with explanations.
- `journalctl -b` isolates one boot session.
- Previous boot sessions can be investigated using `-b -1`, `-b -2`, etc.
- `journalctl -p` filters by Syslog severity level.
- Filters can be combined to reduce investigation noise.
- Efficient troubleshooting is about asking precise questions and finding evidence—not reading every log entry from beginning to end.
