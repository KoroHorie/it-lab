# Lesson 4 - Managing Services with `systemctl`

> **Objective**
>
> Learn how to safely manage Linux services using `systemctl` and understand when to use **start**, **stop**, **restart**, and **reload** in real production environments.

---

# Prerequisites

Before managing a service, always verify its current state.

```bash
systemctl status <service>
```

Example:

```bash
systemctl status chronyd.service
```

---

# 1. Starting a Service

## Command

```bash
systemctl start <service>
```

Example:

```bash
systemctl start httpd.service
```

## Purpose

Starts a service that is currently stopped.

At a high level:

```text
systemctl
    ↓
systemd
    ↓
Locate the unit
    ↓
Read the unit configuration
    ↓
Execute ExecStart=
    ↓
Kernel creates the process
    ↓
systemd supervises the process
```

## Production Workflow

1. Verify the service state.
2. Start the service.
3. Verify it is running.

Example:

```bash
systemctl status httpd
systemctl start httpd
systemctl status httpd
```

---

# 2. Stopping a Service

## Command

```bash
systemctl stop <service>
```

Example:

```bash
systemctl stop httpd.service
```

## Purpose

Stops a running service.

At a high level:

```text
systemctl
    ↓
systemd
    ↓
Locate the unit
    ↓
Transition the service
Running → Stopped
    ↓
Verify the service has stopped
```

## Common Use Cases

- Scheduled maintenance
- Application deployment
- Server shutdown preparation
- Troubleshooting

## Production Workflow

1. Verify the service is running.
2. Stop the service.
3. Verify the service is inactive.
4. Notify the application or operations team.

---

# 3. Restarting a Service

## Command

```bash
systemctl restart <service>
```

Example:

```bash
systemctl restart nginx.service
```

## Purpose

Restarts a service by stopping the current process and starting a new one.

Conceptually:

```text
Running
    ↓
Stopped
    ↓
Running
```

## High-Level Flow

```text
systemctl
    ↓
systemd
    ↓
Locate the unit
    ↓
Stop the service
    ↓
Wait until stopped
    ↓
Start the service again
    ↓
Verify it is running
```

## Why Use Restart Instead of Stop + Start?

Using:

```bash
systemctl restart myapp.service
```

is preferred over:

```bash
systemctl stop myapp.service
systemctl start myapp.service
```

because:

- expresses the intended action clearly
- reduces human error
- minimizes the chance of forgetting to start the service again
- lets systemd manage the entire transition

## Common Use Cases

- Application updates
- Package updates
- Major configuration changes
- Recovering an unhealthy service
- Services that do not support reload

---

# 4. Reloading a Service

## Command

```bash
systemctl reload <service>
```

Example:

```bash
systemctl reload nginx.service
```

## Purpose

Reload tells the **existing running process** to re-read its configuration without stopping the service.

Conceptually:

```text
Running Process
      ↓
Read configuration again
      ↓
Continue running
```

Unlike restart:

- the process stays running
- existing service interruption is minimized
- a new process is **not** created

## Typical Use Cases

- Nginx configuration changes
- Apache configuration changes
- Log configuration updates
- Runtime configuration updates supported by the application

## Important

Not every service supports reload.

A unit must define an **ExecReload=** action.

Otherwise:

```bash
systemctl reload myservice
```

returns an error because systemd has no reload action to execute.

---

# Restart vs Reload

| Restart | Reload |
|----------|---------|
| Stops the service | Keeps the service running |
| Starts a new process | Uses the existing process |
| Higher chance of service interruption | Minimal interruption |
| Used after binary or package updates | Used after configuration changes |
| Works for almost every service | Only works if the service supports reload |

---

# Configuration Changes

Before applying configuration changes, validate the application's configuration whenever possible.

Example (Nginx):

```bash
nginx -t
```

If validation succeeds:

```bash
systemctl reload nginx
```

Then verify:

```bash
systemctl status nginx
```

Professional workflow:

```text
Modify configuration
        ↓
Validate configuration
        ↓
Reload service
        ↓
Verify service
```

---

# Production Troubleshooting Workflow

Every administrative task should follow a structured workflow.

```text
Observe
    ↓
Verify
    ↓
Investigate
    ↓
Determine Root Cause
    ↓
Plan
    ↓
Implement Solution
    ↓
Verify Solution
        │
        ├── Success
        │       ↓
        │ Prevent Future Recurrence
        │
        └── Failed
                ↓
        Investigate Again
                ↓
        Determine Root Cause
                ↓
              Plan
                ↓
      Implement Solution
                ↓
        Verify Solution
```

Never assume a command succeeded.

Always verify the result.

---

# Professional Mindset

A Linux System Administrator should not focus only on commands.

Before changing a service, understand:

- Why is the service being changed?
- What systems depend on it?
- Is there an approved maintenance window?
- Can the change be applied using reload instead of restart?
- Does the application support reload?
- How will the change affect users?

The command is only a small part of the job.

The decision behind the command is what distinguishes a professional system administrator.

---

# Key Takeaways

- Always verify a service before changing its state.
- Verify the result after every state-changing operation.
- Use **start** to start a stopped service.
- Use **stop** for maintenance or controlled shutdowns.
- Use **restart** when a new service instance is required.
- Use **reload** when only configuration changes need to be applied and the service supports it.
- Validate application configuration before reloading whenever possible.
- Think about system architecture and operational impact before making changes.
- Follow a structured troubleshooting workflow instead of guessing.
