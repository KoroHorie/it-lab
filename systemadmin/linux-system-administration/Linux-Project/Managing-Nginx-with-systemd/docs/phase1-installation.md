# Phase 1 — Installing Nginx

---

# Objective

Install the Nginx web server on a fresh CentOS Stream 9 virtual machine and verify that the package and its corresponding systemd service have been installed correctly.

Although this phase focuses on package installation, the primary goal is to understand how Linux services become available to **systemd** after installation.

---

# Scenario

ABC Solutions has provisioned a brand-new CentOS Stream 9 virtual machine.

As the Junior Linux System Administrator, I have been assigned to prepare the server for hosting the company's internal website.

The first task is to install the Nginx web server.

---

# Step 1 — Update the Operating System

Before installing any software, update the package repository and installed packages to ensure the server is using the latest available versions.

```bash
sudo dnf update
sudo dnf upgrade -y
```

or

```bash
sudo dnf update && sudo dnf upgrade -y
```

---

# Why update first?

Updating the system before installing packages helps ensure:

- Latest security patches
- Latest package versions
- Bug fixes
- Dependency compatibility

This is considered a best practice before deploying new software on a Linux server.

---

# Step 2 — Install Nginx

Install the Nginx package.

```bash
sudo dnf install nginx -y
```

The installation performs several actions automatically.

- Downloads the Nginx package
- Installs required dependencies
- Places configuration files under `/etc/nginx`
- Installs the systemd service unit
- Registers the service with systemd

---

# What happened behind the scenes?

Once the installation completed, Nginx became a manageable Linux service.

Before installation:

```text
systemd
    │
    └── nginx.service (does not exist)
```

After installation:

```text
systemd
    │
    └── nginx.service
```

Although the service now exists, it has **not** started automatically.

---

# Step 3 — Verify the Configuration File

Verify that the main configuration file exists.

```bash
ls /etc/nginx/nginx.conf
```

Expected output:

```text
/etc/nginx/nginx.conf
```

This confirms that the package installation created the default configuration.

---

# Step 4 — Verify the Service

Check the current state of the service.

```bash
systemctl status nginx
```

Expected state:

```text
Loaded: loaded
Active: inactive (dead)
```

The exact output may vary slightly depending on the operating system version.

---

# Understanding the Initial Service State

Immediately after installation, the Nginx service is available to systemd but is not yet running.

This means:

- The package is installed.
- The service unit exists.
- systemd recognizes the service.
- The web server has not yet started.

This is the expected behavior on CentOS Stream 9.

---

# Service Analysis

At this point, several observations can already be made.

## Loaded

```text
Loaded: loaded
```

The service unit has been successfully installed and recognized by systemd.

---

## Active

```text
Active: inactive (dead)
```

The web server is currently not running.

No Nginx worker processes have been started.

---

## Enabled

Depending on the distribution, the service may initially appear as disabled.

This simply means the service has not yet been configured to start automatically during system boot.

---

# Production Notes

Installing a package does **not** automatically mean the application is running.

Enterprise Linux distributions often separate:

- Package installation
- Service startup
- Automatic startup during boot

This gives administrators full control over when services begin accepting client requests.

---

# Commands Used

```bash
sudo dnf update

sudo dnf upgrade -y

sudo dnf install nginx -y

ls /etc/nginx/nginx.conf

systemctl status nginx
```

---

# Verification Checklist

- [x] Operating system updated
- [x] Nginx package installed
- [x] Configuration file exists
- [x] systemd recognizes nginx.service
- [x] Service state verified

---

# Lessons Learned

- Installing a package also installs its corresponding systemd service unit.
- systemd can manage a service even when the service is not running.
- Package installation and service startup are separate administrative tasks.
- Verifying installation is an important step before beginning any configuration changes.
