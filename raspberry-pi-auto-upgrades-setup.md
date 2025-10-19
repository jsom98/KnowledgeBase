# ⚙️ Raspberry Pi 5 – Automatic Upgrades, Cleanup & Reboot Setup

---

**Last updated:** [10/19/2025]  
**Author:** Jeffrey Som  
**Tags:** Raspberry Pi, System, Updates, Automation, Maintenance  

---

## 📝 Overview

Want your Raspberry Pi 5 to stay fully updated without manual input?  
This guide sets up **automatic system upgrades**, **automatic cleanup**, and **auto reboot** if required.  
Once configured, your Pi will update itself daily, remove unnecessary packages, and reboot automatically when needed.

---

## 🚀 What You’ll Need

✅ Raspberry Pi 5 running Raspberry Pi OS  
✅ Internet connection  
✅ Terminal access (SSH or direct)  
✅ Sudo privileges  

---

## ⚙️ Step 1 – Update and Upgrade Manually Once

Before enabling automation, make sure your system is current.

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove -y
sudo reboot
```

---

## 🧩 Step 2 – Install the Unattended Upgrades Package

Install and enable the unattended-upgrades package:

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

This configures automatic updates for security and system packages.

---

## 🛠️ Step 3 – Configure Automatic Updates and Cleanup

Edit the unattended-upgrades configuration file:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Find and modify/add these lines (uncomment or add if missing):

```
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:30";
```

Explanation:
- `"Remove-Unused-Dependencies"` → Removes unneeded packages automatically  
- `"Automatic-Reboot"` → Reboots when required  
- `"Automatic-Reboot-Time"` → Reboots at 2:30 AM  

Save and exit (`Ctrl + O`, `Enter`, `Ctrl + X`).

---

## 🔧 Step 4 – Set Up Daily Update Schedule

Edit the auto-update configuration file:

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

Replace its contents with:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

Explanation:
- Updates daily  
- Downloads upgrades daily  
- Cleans weekly  
- Automatically applies upgrades  

Save and exit.

---

## 🔄 Step 5 – Test Automatic Upgrades

Run a dry test to confirm your setup:

```bash
sudo unattended-upgrades --dry-run --debug
```

You should see simulated upgrade actions, cleanup, and reboot messages.

---

## 🧹 Step 6 – Create a Daily Cleanup Script

To ensure old packages are always removed, create this cleanup script:

```bash
sudo nano /usr/local/bin/auto-cleanup.sh
```

Paste this inside:

```bash
#!/bin/bash
apt autoremove -y
apt autoclean -y
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/auto-cleanup.sh
```

Then add it to cron to run daily:

```bash
sudo crontab -e
```

Add this line at the bottom:

```
30 3 * * * /usr/local/bin/auto-cleanup.sh
```

This runs cleanup every day at **3:30 AM** (after unattended upgrades and reboots if needed).

---

## 🧠 Step 7 – One-Click Auto Setup Script (Optional)

If you want to install and configure everything automatically, create this one-click script:

```bash
sudo nano auto-update-setup.sh
```

Paste the following:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Usage:
#   sudo bash auto-update-setup.sh
#   sudo bash auto-update-setup.sh --enable-rpi-firmware   # optional: adds weekly rpi-update cron

require_root() {
  if [[ "${EUID}" -ne 0 ]]; then
    echo "Please run as root: sudo bash $0"
    exit 1
  fi
}

info() { echo -e "\n[INFO] $*"; }
ok()   { echo "[OK] $*"; }
warn() { echo "[WARN] $*"; }

require_root

ENABLE_RPI_FW="no"
if [[ "${1:-}" == "--enable-rpi-firmware" ]]; then
  ENABLE_RPI_FW="yes"
fi

CODENAME="$(. /etc/os-release && echo "${VERSION_CODENAME:-$(lsb_release -c -s 2>/dev/null || true)}")"
if [[ -z "${CODENAME}" ]]; then
  warn "Could not determine VERSION_CODENAME. Using unattended-upgrades builtin \${distro_codename} variable."
else
  ok "Detected codename: ${CODENAME}"
fi

info "Installing unattended-upgrades…"
apt-get update
apt-get install -y unattended-upgrades

CONF_50="/etc/apt/apt.conf.d/50unattended-upgrades"
info "Configuring ${CONF_50}…"
cat > "${CONF_50}" <<'EOF'
// Automatically install security and OS updates for Raspberry Pi OS

Unattended-Upgrade::Origins-Pattern {
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
        "origin=Raspberry Pi Foundation";
};

Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:30";
Unattended-Upgrade::Verbose "true";
Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::SyslogFacility "daemon";
EOF
ok "Wrote ${CONF_50}"

CONF_20="/etc/apt/apt.conf.d/20auto-upgrades"
info "Configuring ${CONF_20}…"
cat > "${CONF_20}" <<'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOF
ok "Wrote ${CONF_20}"

info "Enabling unattended-upgrades service and APT timers…"
systemctl enable --now unattended-upgrades
systemctl enable --now apt-daily.timer apt-daily-upgrade.timer || true
ok "Services/timers enabled"

if [[ "${ENABLE_RPI_FW}" == "yes" ]]; then
  if command -v rpi-update >/dev/null 2>&1; then
    info "Adding weekly firmware update cron (Sunday 03:00)…"
    TMP_CRON="$(mktemp)"
    { crontab -l 2>/dev/null || true; } > "${TMP_CRON}"
    sed -i '\#/usr/bin/rpi-update && /usr/bin/apt autoremove -y && /usr/bin/apt autoclean -y && /bin/systemctl reboot#d' "${TMP_CRON}"
    echo '0 3 * * 0 /usr/bin/rpi-update && /usr/bin/apt autoremove -y && /usr/bin/apt autoclean -y && /bin/systemctl reboot' >> "${TMP_CRON}"
    crontab "${TMP_CRON}"
    rm -f "${TMP_CRON}"
    ok "Weekly firmware cron installed"
  else
    warn "rpi-update not found. Install it if you want bleeding-edge firmware: apt install rpi-update"
  fi
fi

info "Running verification (dry-run)…"
unattended-upgrades --dry-run --debug || true

echo
ok "unattended-upgrades service:"
systemctl --no-pager status unattended-upgrades | sed -n '1,10p' || true

echo
ok "APT timers (next runs):"
systemctl list-timers | grep -E 'apt-daily|apt-daily-upgrade' || true

echo
ok "If a reboot is required, /var/run/reboot-required will exist and the Pi will reboot at 02:30."
ok "Logs: /var/log/unattended-upgrades/unattended-upgrades.log"
echo
ok "Setup complete."
```

Make it executable and run it:

```bash
sudo chmod +x auto-update-setup.sh
sudo ./auto-update-setup.sh
```

Or, to include **weekly firmware updates** too:

```bash
sudo ./auto-update-setup.sh --enable-rpi-firmware
```

This script does everything automatically — installs, configures, verifies, and enables maintenance for you.

---

## 🔍 Step 8 – Verify Automation

Check timers:
```bash
systemctl list-timers | grep apt
```

View the unattended-upgrades log:
```bash
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```

You should see successful entries for updates, cleanups, and reboots.

---

## ✅ Done!

Your Raspberry Pi 5 is now fully automated:

- 🧩 Updates daily  
- 🧹 Cleans unused packages  
- 🔁 Reboots when needed  
- ⚡ Optional one-click setup script for fast deployment  

You can leave your Pi running — it will handle updates, maintenance, and reboots automatically.
