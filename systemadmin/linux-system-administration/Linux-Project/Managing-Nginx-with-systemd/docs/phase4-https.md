# Phase 4 — Securing the Web Server with HTTPS

---

# Objective

Configure Nginx to securely serve the website over HTTPS by generating a self-signed SSL/TLS certificate, configuring Nginx to use the certificate, validating the configuration, and allowing HTTPS traffic through the Linux firewall.

This phase introduces the fundamentals of SSL/TLS deployment on Linux while reinforcing safe service management using **systemd** and **systemctl**.

---

# Scenario

The website is now accessible over HTTP.

To improve security, the Infrastructure Team has requested that the internal web application also support encrypted HTTPS communication.

Since this is an internal laboratory environment, a **self-signed certificate** will be used instead of a certificate issued by a trusted Certificate Authority.

---

# SSL/TLS Overview

HTTP sends data across the network in plaintext.

HTTPS encrypts communication between the client and the web server using SSL/TLS.

This protects:

- User credentials
- Session cookies
- Application data
- Internal company information

Even though this project hosts a static webpage, configuring HTTPS introduces the same deployment workflow used in production environments.

---

# Step 1 — Create the SSL Directory

Create a dedicated directory to store SSL certificates.

```bash
sudo mkdir -p /etc/nginx/ssl
```

Verify:

```bash
ls -ld /etc/nginx/ssl
```

---

# Why Create a Separate Directory?

Separating SSL assets from the rest of the Nginx configuration improves organization.

Typical production layout:

```text
/etc/nginx/

├── nginx.conf
├── conf.d/
├── ssl/
│     ├── nginx.crt
│     └── nginx.key
```

Keeping certificates in their own directory simplifies administration and backups.

---

# Step 2 — Generate a Self-Signed Certificate

Generate a private key and self-signed certificate.

```bash
sudo openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/nginx/ssl/nginx.key \
-out /etc/nginx/ssl/nginx.crt
```

Example values:

```text
Country Name: PH
Organization Name: ABC Solutions
Organizational Unit: IT Department
Common Name: <Server-IP>
```

---

# Verify

```bash
ls -l /etc/nginx/ssl
```

Expected:

```text
nginx.crt
nginx.key
```

---

# Step 3 — Create an HTTPS Server Configuration

Instead of modifying the main `nginx.conf`, create a separate configuration file.

```bash
sudo vi /etc/nginx/conf.d/ssl.conf
```

Configuration:

```nginx
server {

    listen 443 ssl;
    listen [::]:443 ssl;

    server_name <Server-IP>;

    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate     /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    location / {
        try_files $uri $uri/ =404;
    }

}
```

Replace:

```text
<Server-IP>
```

with the IP address of the CentOS server.

---

# Why Create a Separate ssl.conf?

Instead of modifying `nginx.conf`, HTTPS was configured in a dedicated server block.

Benefits:

- Easier maintenance
- Cleaner configuration
- Modular design
- Easier troubleshooting

This organization is commonly used in production environments.

---

# Step 4 — Validate the Configuration

Before applying configuration changes:

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

---

# Why Validate First?

Applying an invalid configuration can prevent Nginx from reloading successfully.

Validation allows configuration errors to be detected before they impact the running service.

This is considered a standard production practice.

---

# Step 5 — Reload the Service

Since only the configuration changed, reload the service.

```bash
sudo systemctl reload nginx
```

Verify:

```bash
systemctl status nginx
```

Expected:

```text
Active: active (running)
```

---

# Why Reload Instead of Restart?

Reload instructs Nginx to re-read its configuration without terminating the running worker processes.

Benefits:

- Existing client connections remain active.
- Minimal service interruption.
- Faster than a full restart.

Whenever supported, **reload** is generally preferred for configuration-only changes.

---

# Step 6 — Verify Listening Ports

Verify that Nginx is now listening on both HTTP and HTTPS.

```bash
sudo ss -tlnp | grep nginx
```

Expected:

```text
LISTEN ... :80
LISTEN ... :443
```

This confirms that Nginx has successfully opened the HTTPS listener.

---

# Step 7 — Allow HTTPS Through firewalld

Allow HTTPS traffic.

```bash
sudo firewall-cmd --add-service=https
```

Verify:

```bash
sudo firewall-cmd --list-all
```

Expected:

```text
services: ssh http https
```

After successful verification:

```bash
sudo firewall-cmd --runtime-to-permanent
```

---

# Runtime vs Permanent Configuration

During implementation, an important distinction between runtime and permanent firewall configurations was observed.

Runtime changes:

```bash
firewall-cmd --add-service=https
```

take effect immediately.

Permanent changes:

```bash
firewall-cmd --permanent --add-service=https
```

modify the saved configuration but do **not** affect the currently running firewall until it is reloaded.

A safer workflow is:

```text
Modify Runtime
        ↓
Verify
        ↓
Save Runtime to Permanent
```

using:

```bash
firewall-cmd --runtime-to-permanent
```

This reduces the risk of permanently saving an incorrect firewall configuration.

---

# Step 8 — Verify HTTPS

Test locally.

```bash
curl -vk https://localhost
```

or

```bash
curl -vk https://<Server-IP>
```

The `-k` option tells curl to ignore certificate validation because the certificate is self-signed.

---

# Step 9 — Verify From Another Machine

Open:

```text
https://<Server-IP>
```

Expected:

- Browser displays a security warning.
- Continue to the website.
- The webpage loads successfully over HTTPS.

The warning is expected because the certificate is not issued by a trusted Certificate Authority.

---

# Troubleshooting Performed

Several issues were encountered during deployment.

---

## Issue 1

```text
nginx: [emerg] no "ssl_certificate" is defined...
```

### Root Cause

The HTTPS server block was incomplete during the initial configuration.

### Resolution

Completed the SSL configuration and validated it using:

```bash
sudo nginx -t
```

---

## Issue 2

HTTPS traffic still failed.

Investigation showed:

```bash
ss -tlnp | grep nginx
```

Initially, only port 80 was observed.

Further verification confirmed the HTTPS configuration had not yet been fully applied.

After reloading the service, Nginx successfully began listening on port 443.

---

## Issue 3

HTTPS remained inaccessible from remote clients.

The firewall configuration was investigated.

Initially:

```bash
firewall-cmd --permanent --add-service=https
```

did not immediately expose HTTPS because the runtime firewall remained unchanged.

Allowing HTTPS during runtime:

```bash
firewall-cmd --add-service=https
```

resolved the issue immediately.

The runtime configuration was then committed permanently using:

```bash
firewall-cmd --runtime-to-permanent
```

---

# Commands Used

```bash
mkdir -p /etc/nginx/ssl

openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/nginx/ssl/nginx.key \
-out /etc/nginx/ssl/nginx.crt

vi /etc/nginx/conf.d/ssl.conf

nginx -t

systemctl reload nginx

ss -tlnp | grep nginx

firewall-cmd --add-service=https

firewall-cmd --list-all

firewall-cmd --runtime-to-permanent

curl -vk https://localhost
```

---

# Verification Checklist

- [x] SSL directory created.
- [x] Self-signed certificate generated.
- [x] HTTPS server block configured.
- [x] Configuration validated.
- [x] Nginx reloaded successfully.
- [x] Port 443 verified.
- [x] HTTPS allowed through firewalld.
- [x] HTTPS verified locally.
- [x] HTTPS verified remotely.

---

# Lessons Learned

- HTTPS requires both a certificate and its corresponding private key.
- Nginx should always be validated using `nginx -t` before reloading.
- Configuration-only changes should generally use `systemctl reload`.
- Verifying listening ports confirms that a service has successfully bound to its expected network ports.
- firewalld maintains separate runtime and permanent configurations.
- Runtime changes should be verified before being saved permanently.
- Structured troubleshooting is more effective than making random configuration changes.

---

# Project Completion

This project successfully demonstrated the complete deployment lifecycle of an Nginx web server on CentOS Stream 9.

The project included:

- Installing software
- Managing services with systemd
- Deploying a website
- Troubleshooting connectivity issues
- Configuring HTTPS
- Managing firewall rules
- Following a structured troubleshooting methodology
- Documenting the implementation

This marks the completion of **Project 01 — Managing Nginx with systemd and systemctl**, providing a strong foundation for future Linux System Administration projects involving reverse proxies, virtual hosts, application servers, and enterprise web infrastructure.
