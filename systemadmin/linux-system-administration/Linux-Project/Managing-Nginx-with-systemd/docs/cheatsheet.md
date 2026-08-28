# Nginx + systemd Cheat Sheet

> Project 01 — Managing Nginx with systemd and systemctl

---

# Service Management

Check service status

```bash
systemctl status nginx
```

Start service

```bash
sudo systemctl start nginx
```

Stop service

```bash
sudo systemctl stop nginx
```

Restart service

```bash
sudo systemctl restart nginx
```

Reload configuration

```bash
sudo systemctl reload nginx
```

Enable service during boot

```bash
sudo systemctl enable nginx
```

Disable automatic startup

```bash
sudo systemctl disable nginx
```

---

# Inspecting the Service

View the unit file

```bash
systemctl cat nginx
```

View recent logs

```bash
journalctl -u nginx
```

Latest logs

```bash
journalctl -u nginx -n 20
```

Follow logs live

```bash
journalctl -fu nginx
```

---

# Process Verification

Find running processes

```bash
ps -ef | grep nginx
```

Display specific process

```bash
ps -fp <PID>
```

Verify parent process

```bash
ps -o pid,ppid,cmd -p <PID>
```

---

# Verify Listening Ports

Display listening ports

```bash
ss -tlnp
```

Display only nginx

```bash
ss -tlnp | grep nginx
```

Display HTTPS

```bash
ss -tlnp | grep :443
```

Display HTTP

```bash
ss -tlnp | grep :80
```

---

# Test the Website

Local HTTP

```bash
curl -i http://localhost
```

Server IP

```bash
curl -i http://<Server-IP>
```

HTTPS

```bash
curl -vk https://localhost
```

---

# Validate Configuration

Always validate before reload or restart.

```bash
nginx -t
```

Dump the complete running configuration

```bash
nginx -T
```

---

# Logs

Access Log

```bash
tail -f /var/log/nginx/access.log
```

Error Log

```bash
tail -f /var/log/nginx/error.log
```

---

# Firewall

Display configuration

```bash
firewall-cmd --list-all
```

Allow HTTP

```bash
firewall-cmd --add-service=http
```

Allow HTTPS

```bash
firewall-cmd --add-service=https
```

Save runtime configuration

```bash
firewall-cmd --runtime-to-permanent
```

Reload permanent configuration

```bash
firewall-cmd --reload
```

---

# SSL

Generate a self-signed certificate

```bash
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/nginx/ssl/nginx.key \
-out /etc/nginx/ssl/nginx.crt
```

Inspect certificate

```bash
openssl x509 -in /etc/nginx/ssl/nginx.crt -text -noout
```

---

# Typical Administrative Workflow

```text
Install
    ↓
Configure
    ↓
Validate
    ↓
Reload
    ↓
Verify
```

---

# Troubleshooting Workflow

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

---

# Reload vs Restart

Use **reload** when:

- Configuration changed
- Application supports reload
- Existing connections should remain active

Examples

- Nginx
- Apache
- HAProxy

---

Use **restart** when:

- Reload is unsupported
- Service is hung
- Binary updated
- Libraries updated
- Kernel interaction changed
- Service failed to reload

---

# Before Reloading

Always perform:

```bash
nginx -t
```

Never reload an unvalidated configuration.

---

# Before Restarting

Ask yourself:

- Why am I restarting?
- Is reload sufficient?
- Will users be affected?
- Is a maintenance window required?
- Have I validated my configuration?

---

# Production Mindset

Never assume.

Always verify.

Never change production blindly.

Collect evidence first.

Implement the smallest possible change.

Verify the result.

Document what happened.
