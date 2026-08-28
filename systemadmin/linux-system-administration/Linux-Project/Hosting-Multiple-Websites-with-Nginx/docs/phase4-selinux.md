# Phase 4 — Troubleshooting SELinux Web Access Restrictions

## Objective

The objective of this phase is to troubleshoot and resolve an Nginx access issue caused by SELinux security policies.

During testing, the websites returned:

```
403 Forbidden
```

Although:

- Nginx configuration was correct.
- Website files existed.
- File ownership appeared correct.
- Unix permissions allowed reading.

The investigation required understanding how SELinux works together with traditional Linux permissions.

---

# Production Scenario

ABC Solutions has successfully configured multiple HTTPS websites.

However, after deploying the websites, users are unable to access them.

Browser response:

```
403 Forbidden
```

The administrator must investigate why Nginx cannot serve the website files.

---

# Incident Report

## User Impact

Users attempting to access:

```
https://company.local

https://hr.company.local

https://it.company.local
```

receive:

```
403 Forbidden
```

---

# Troubleshooting Workflow

The incident followed the standard troubleshooting process:

```
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

---

# Step 1 — Verify Nginx Status

First, verify that Nginx is running.

Command:

```bash
systemctl status nginx
```

Result:

```
active (running)
```

Conclusion:

The service itself is available.

The issue is not caused by Nginx being stopped.

---

# Step 2 — Verify Website Files Exist

Check the website directory:

```bash
ls -l /var/www/company
```

Result:

```
-rw-r--r-- root root index.html
```

The file exists.

---

# Step 3 — Verify Unix Permissions

Check directory permissions:

```bash
ls -ld /var/www

ls -ld /var/www/company

ls -l /var/www/company
```

Result:

```
drwxr-xr-x root root /var/www

drwxr-xr-x root root /var/www/company

-rw-r--r-- root root index.html
```

Analysis:

The permissions allow read access.

Nginx should technically be able to read the files.

---

# Understanding the Problem

At this point, the question becomes:

> If Linux permissions allow access, why is Nginx still blocked?

Linux has another security layer:

```
Traditional Permissions
        |
        ↓
Discretionary Access Control (DAC)


SELinux
        |
        ↓
Mandatory Access Control (MAC)
```

Both security systems must allow access.

---

# Step 4 — Investigating SELinux Contexts

Check the SELinux labels:

Command:

```bash
ls -Zd /var/www/company
```

Result:

```
unconfined_u:object_r:var_t:s0
```

Compare with the default Nginx web directory:

Command:

```bash
ls -Zd /usr/share/nginx/html/
```

Result:

```
system_u:object_r:httpd_sys_content_t:s0
```

---

# Understanding the Difference

The important part is the SELinux type:

Current website:

```
var_t
```

Default Nginx content:

```
httpd_sys_content_t
```

SELinux does not only care about file permissions.

It also checks:

```
Who is accessing?

What process is accessing?

What type is the object?
```

---

# Root Cause Analysis

The website files were labeled as:

```
var_t
```

SELinux interpreted these files as general variable data.

The Nginx process runs under the HTTP daemon security policy.

The policy allows Nginx to read web content labeled:

```
httpd_sys_content_t
```

but not arbitrary files labeled:

```
var_t
```

Therefore:

```
Nginx

        |
        ↓

Kernel Access Request

        |
        ↓

SELinux Policy Check

        |
        ↓

var_t detected

        |
        ↓

Access denied

        |
        ↓

Nginx returns 403 Forbidden
```

---

# Step 5 — Applying the Solution

The website directory must receive the correct SELinux context.

First, define the correct label:

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www(/.*)?"
```

Then apply the label:

```bash
restorecon -Rv /var/www
```

Output:

```
Relabeled /var/www/company
from var_t
to httpd_sys_content_t
```

---

# Verification

Check the SELinux context again:

```bash
ls -Zd /var/www/company
```

Result:

```
unconfined_u:object_r:httpd_sys_content_t:s0
```

The website directory now has the correct SELinux label.

---

# Testing Website Access

Access:

```
https://company.local
```

Result:

```
ABC Solutions

Welcome to Company Portal
```

The 403 Forbidden error is resolved.

---

# Understanding DAC and SELinux Together

This incident demonstrated that Linux access control happens through multiple layers.

Example:

```
User Request

      |
      ↓

Linux Permissions Check

      |
      ↓

SELinux Policy Check

      |
      ↓

Kernel Decision

      |
      ↓

Application Response
```

Both layers must allow access.

---

# Important Lesson

Changing ownership or permissions is not always the solution.

A common administrator mistake is:

```
chmod 777
```

when troubleshooting access problems.

However, if SELinux blocks access, permissions will not solve the problem.

The correct approach:

1. Check permissions.
2. Check SELinux context.
3. Check logs.
4. Apply the correct security configuration.

---

# Verification Checklist

| Check | Result |
|-|-|
| Nginx running | ✅ |
| Website files exist | ✅ |
| Unix permissions verified | ✅ |
| SELinux context identified | ✅ |
| Correct SELinux label applied | ✅ |
| Website access restored | ✅ |

---

# Production Considerations

In production environments:

Administrators should avoid disabling SELinux as a quick fix.

Bad approach:

```bash
setenforce 0
```

Better approach:

- Understand the denial.
- Assign correct contexts.
- Create proper SELinux rules if required.

SELinux provides an additional security boundary that protects services even when traditional permissions are incorrectly configured.

---

# Key Takeaways

This phase demonstrated real Linux troubleshooting beyond basic permissions.

Important concepts learned:

- Unix permissions are not the only access control mechanism.
- SELinux provides Mandatory Access Control.
- File contexts determine how services can interact with objects.
- Nginx requires appropriate SELinux labels to serve web content.
- Production troubleshooting requires investigation before modification.

The next phase will configure HTTP → HTTPS redirection so users are automatically moved to secure communication.
