# Phase 2 — Configuring and Managing Nginx with systemd

---

# Objective

Configure the default Nginx website, start the web server, enable automatic startup during system boot, and verify that the website is successfully being served to clients.

This phase introduces the practical use of **systemctl** for managing Linux services and reinforces the relationship between **systemd**, **service units**, and the actual application process.

---

# Scenario

The Nginx package has already been installed successfully.

The Infrastructure Team has provided a simple static HTML page that will be hosted on the server.

As the Junior Linux System Administrator, my responsibilities are:

- Configure the website.
- Verify the current service state.
- Understand how the service is configured.
- Start the service.
- Enable the service during boot.
- Verify the website can be accessed.

---

# Step 1 — Deploy the Website

Replace the default web page with a simple HTML page.

Example:

```bash
sudo vi /usr/share/nginx/html/index.html
```

Example content:

```html
<!DOCTYPE html>

<html>

<head>
    <title>ABC Solutions</title>
</head>

<body>

<h1>ABC Solutions</h1>

<p>Welcome to our Linux Web Server.</p>

</body>

</html>
```

Save the file.

---

# Why configure the website first?

Before starting a web server, it is generally good practice to ensure that the content it will serve is already prepared.

This prevents users from seeing placeholder pages or incomplete content once the service becomes available.

---

# Step 2 — Verify the Current Service State

Before making any changes, inspect the current state of the service.

```bash
systemctl status nginx
```

Observed state:

```text
Loaded: loaded
Active: inactive (dead)
```

Analysis:

- The service unit exists.
- systemd recognizes the service.
- The web server is currently not running.

---

# Step 3 — Inspect the Service Unit

View the service definition managed by systemd.

```bash
systemctl cat nginx
```

---

# Unit Section

Example:

```ini
[Unit]

Description=The nginx HTTP and reverse proxy server

After=network-online.target remote-fs.target nss-lookup.target

Wants=network-online.target
```

Observation:

The service is designed to start after the required system resources become available.

The **After=** directive specifies the startup order.

The **Wants=** directive expresses a soft dependency on the network becoming available.

---

# Service Section

Example:

```ini
ExecStartPre=/usr/bin/rm -f /run/nginx.pid

ExecStartPre=/usr/sbin/nginx -t

ExecStart=/usr/sbin/nginx

ExecReload=/usr/sbin/nginx -s reload
```

Observations:

Before Nginx starts, systemd performs two preparation steps.

First, any stale PID file is removed.

Second, the Nginx configuration is validated.

Only after these checks succeed does systemd execute the Nginx binary.

The presence of **ExecReload** indicates that Nginx supports configuration reloads without requiring a full restart.

---

# Install Section

Example:

```ini
WantedBy=multi-user.target
```

Observation:

This service can be enabled to start automatically whenever the system enters the **multi-user.target**.

The service is not automatically enabled simply because it has been installed.

---

# Step 4 — Start the Service

Start Nginx.

```bash
sudo systemctl start nginx
```

---

# What happened internally?

The request follows this sequence:

```text
systemctl
        │
        ▼
systemd (PID 1)
        │
        ▼
nginx.service
        │
        ▼
ExecStart
        │
        ▼
/usr/sbin/nginx
        │
        ▼
Linux Kernel
        │
        ▼
nginx process starts
```

The Linux kernel ultimately creates the running process.

systemd becomes the parent process and continuously supervises the service.

---

# Step 5 — Verify the Service

```bash
systemctl status nginx
```

Expected:

```text
Active: active (running)
```

The status output should also display:

- Main PID
- Memory usage
- Running tasks
- Recent service logs

---

# Step 6 — Verify the Running Process

Confirm that the Nginx process exists.

```bash
ps -ef | grep nginx
```

or

```bash
ps -fp <PID>
```

The Main PID displayed by **systemctl status** should match the running process.

The parent process (PPID) should be:

```text
1
```

indicating that the process is supervised by **systemd**.

---

# Step 7 — Enable the Service

Configure Nginx to start automatically whenever the server boots.

```bash
sudo systemctl enable nginx
```

---

# Verify

```bash
systemctl status nginx
```

Expected:

```text
Loaded: loaded (...; enabled; ...)
```

The service is now associated with **multi-user.target**.

---

# Step 8 — Verify Website Locally

Test the web server from the server itself.

```bash
curl -i http://localhost
```

Expected:

```text
HTTP/1.1 200 OK
```

The HTML content configured earlier should be returned.

---

# Step 9 — Verify from Another Machine

Open a browser on another machine.

Navigate to:

```text
http://<Server-IP>
```

Expected result:

The webpage should load successfully.

---

# Actual Result During the Lab

The webpage did **not** load.

The browser failed to establish an HTTP connection.

No requests appeared inside:

```text
/var/log/nginx/access.log
```

This indicated that the HTTP request never reached Nginx.

At this point, the project transitioned into a production-style troubleshooting exercise.

The investigation is documented in **Phase 3**.

---

# Commands Used

```bash
systemctl status nginx

systemctl cat nginx

sudo systemctl start nginx

sudo systemctl enable nginx

systemctl status nginx

ps -ef | grep nginx

curl -i http://localhost
```

---

# Verification Checklist

- [x] Website deployed
- [x] Service state verified
- [x] Service unit inspected
- [x] Nginx started successfully
- [x] Running process verified
- [x] Service enabled during boot
- [x] Local HTTP test successful
- [x] Remote HTTP test failed (leading to troubleshooting)

---

# Lessons Learned

- Always inspect the current service state before making changes.
- A service unit provides valuable information about how an application is started and managed.
- `ExecStartPre` allows systemd to perform validation before launching the application.
- `systemctl start` requests systemd to execute the service's `ExecStart` directive.
- The Linux kernel creates the actual running process, while systemd supervises it.
- Verifying the process confirms that the service is truly running.
- Local connectivity does not guarantee remote connectivity.
- When a remote client cannot access the service, begin a structured troubleshooting process rather than making assumptions.
