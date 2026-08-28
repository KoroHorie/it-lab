# VMware Workstation Pro 17.6.4 Installation on Fedora 44

Follow these steps to install VMware Workstation Pro 17.6.4 on Fedora 44.

---

## 1. Update and Upgrade System
```bash
sudo dnf update && sudo dnf upgrade -y
```

---

## 2. Install Required Packages
```bash
sudo dnf install kernel-headers kernel-devel gcc gcc-c++ make libaio
sudo reboot
```

---

## 3. Download VMware Workstation Pro

1. Go to [Broadcom](https://www.broadcom.com/) and create an account.
2. Choose **VMware Cloud Foundation**.
3. Navigate to **My Downloads**.
4. Below the dropdown menu, click **Free Software Downloads**.
5. Look for **VMware Workstation Pro**, choose your version, and download it.

---

## 4. Verify Download
Open Terminal, navigate to the `Downloads` folder, and check the file:
```bash
cd ~/Downloads
ls
```
Ensure `VMware-Workstation-Full-*.bundle` is present.

---

## 5. Clone Host Modules Repository
```bash
git clone https://github.com/Technogeezer50/vmware-host-modules.git
cd vmware-host-modules
```

---

## 6. Build and Install Host Modules
```bash
git checkout workstation-17.6.4
make
sudo make install
```

---

## 7. Restart VMware Services
```bash
sudo systemctl restart vmware
```

---

## 8. Launch VMware Workstation Installer
```bash
vmware
```
The installer will pop up. Follow the prompts to complete the installation of VMware Workstation Pro 17.

---

*You are now ready to use VMware Workstation Pro 17.6.4 on Fedora 44!*

