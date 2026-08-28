# Phase 3 — Troubleshooting Website Connectivity

---

# Objective

Investigate and resolve an incident where remote users are unable to access the Nginx web server despite the service appearing to run normally.

This phase introduces a structured troubleshooting methodology commonly used by Linux System Administrators in production environments.

Rather than immediately applying fixes, every action is supported by evidence gathered during the investigation.

---

# Scenario

After successfully installing, configuring, and starting the Nginx web server, local verification confirmed that the service was operational.

However, users attempting to access the website from another machine reported that the webpage could not be reached.

As the Linux System Administrator, the objective is to identify the root cause and restore service availability.

---

# Troubleshooting Methodology

Throughout this project, every incident follows the same workflow.

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
    ↓
Prevent Future Recurrence
```

This methodology minimizes unnecessary changes and encourages evidence-based troubleshooting.

---

# Step 1 — Observe

The first report from users was:

> "The website cannot be accessed."

No assumptions were made.

At this stage, only the symptom is known.

---

# Step 2 — Verify

## Verify the Nginx Service

```bash
systemctl status nginx
```

Result:

```text
Active: active (running)
```

The web server was confirmed to be running normally.

---

## Verify Locally

Test the website directly from the server.

```bash
curl -i http://localhost
```

Result:

```text
HTTP/1.1 200 OK
```

The website was successfully returned.

The same result was observed using the server's IP address.

```bash
curl -i http://<Server-IP>
```

Result:

```text
HTTP/1.1 200 OK
```

Conclusion:

Nginx itself was functioning correctly.

---

# Step 3 — Investigate

Since local requests succeeded while remote requests failed, the investigation focused on determining where communication stopped.

---

## Monitor the Access Log

Observe incoming HTTP requests.

```bash
tail -f /var/log/nginx/access.log
```

Generate an HTTP request from another machine.

Result:

No log entries appeared.

---

## Generate a Local Request

```bash
curl http://localhost
```

Immediately after executing the command, the access log recorded the request.

Conclusion:

Local requests successfully reached Nginx.

Remote requests never reached Nginx.

---

# Investigation Summary

Evidence collected:

| Test | Result |
|-------|--------|
| Nginx Service | Running |
| Local HTTP Request | Success |
| Remote HTTP Request | Failed |
| Local Access Log | Generated |
| Remote Access Log | No Entry |

The investigation indicated that Nginx itself was not responsible for the issue.

Traffic was being blocked before reaching the web server.

---

# Step 4 — Determine Root Cause

Based on the collected evidence, the most probable root cause was:

> The Linux firewall was blocking incoming HTTP traffic.

---

# Step 5 — Plan

Before modifying the firewall, verify which firewall management utility is active.

The server uses:

```text
firewalld
```

The plan was:

1. Verify firewall configuration.
2. Allow HTTP traffic.
3. Test website accessibility.
4. Confirm successful communication.

---

# Step 6 — Implement Solution

Allow HTTP through firewalld.

```bash
sudo firewall-cmd --add-service=http
```

Verify:

```bash
sudo firewall-cmd --list-all
```

Expected:

```text
services: ssh http
```

Once verified, save the configuration permanently.

```bash
sudo firewall-cmd --runtime-to-permanent
```

---

# Why Use Runtime First?

Instead of immediately modifying the permanent firewall configuration, the runtime configuration was updated first.

This approach allows administrators to verify that the change resolves the issue before committing it permanently.

If the runtime modification introduces unexpected behavior, it can simply be discarded.

Only after successful verification should the runtime configuration be saved permanently.

This is considered a safer production practice.

---

# Step 7 — Verify Solution

From another machine:

```
http://<Server-IP>
```

Result:

The webpage loaded successfully.

---

## Verify Through Access Log

Continue monitoring:

```bash
tail -f /var/log/nginx/access.log
```

Generate another HTTP request.

Result:

A new access log entry appeared immediately.

Example:

```text
192.168.1.100 - - [date] "GET / HTTP/1.1" 200
```

This confirmed that remote traffic was now successfully reaching Nginx.

---

# Step 8 — Prevent Future Recurrence

Document the firewall configuration.

Future deployments should include firewall verification as part of the web server deployment checklist.

Recommended deployment sequence:

```text
Install
    ↓
Configure
    ↓
Start Service
    ↓
Verify Local Access
    ↓
Verify Firewall Rules
    ↓
Verify Remote Access
```

Following this sequence reduces the likelihood of repeating the same issue during future deployments.

---

# Commands Used

```bash
systemctl status nginx

curl -i http://localhost

curl -i http://<Server-IP>

tail -f /var/log/nginx/access.log

firewall-cmd --add-service=http

firewall-cmd --list-all

firewall-cmd --runtime-to-permanent
```

---

# Verification Checklist

- [x] Verified Nginx was running.
- [x] Verified local HTTP connectivity.
- [x] Determined remote clients could not reach the server.
- [x] Used access logs as supporting evidence.
- [x] Identified firewalld as the root cause.
- [x] Allowed HTTP traffic.
- [x] Verified remote access.
- [x] Saved the runtime firewall configuration permanently.

---

# Lessons Learned

- Never assume the application is responsible for every connectivity issue.
- Always distinguish between local and remote connectivity.
- Logs provide valuable evidence during troubleshooting.
- Root cause analysis should be based on observed evidence rather than assumptions.
- firewalld separates runtime and permanent configurations.
- Runtime changes should be verified before committing them permanently.
- Following a structured troubleshooting workflow leads to faster and more reliable incident resolution.
