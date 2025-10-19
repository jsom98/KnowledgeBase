# 🛑 Raspberry Pi 5 – Disable Automatic Upgrades, Cleanup & Reboot Setup

---

**Last updated:** [10/19/2025]  
**Author:** Jeffrey Som  
**Tags:** Raspberry Pi, System, Updates, Automation, Maintenance, Disable  

---

## 📝 Overview

Need to regain full manual control over your Raspberry Pi 5’s update process?  
This guide walks you through how to **disable all automatic updates, cleanups, and reboots** that were previously configured.  

It’s useful for:
- 🔍 Troubleshooting update-related issues  
- 🧩 Testing new configurations manually  
- ⚙️ Preventing reboots during critical workloads  

After completing this, your Raspberry Pi 5 will **no longer update, reboot, or clean automatically**.

---

## 🚀 What You’ll Need

✅ Raspberry Pi 5 running Raspberry Pi OS  
✅ Terminal access (SSH or direct)  
✅ Sudo privileges  

---

## ⚙️ Step 1 – Disable the Unattended-Upgrades Service

Stop and disable the unattended-upgrades service:

```bash
sudo systemctl stop unattended-upgrades
sudo systemctl disable unattended-upgrades
```

Then mask it to prevent it from being started manually or by other services:

```bash
sudo systemctl mask unattended-upgrades
```

Confirm it’s disabled:

```bash
sudo systemctl status unattended-upgrades
```

You should see **“Loaded: masked”** or **“inactive (dead)”**.

---

## 🔧 Step 2 – Disable APT Daily Timers

APT timers control daily checks and upgrades in the background. Disable them:

```bash
sudo systemctl disable --now apt-daily.timer apt-daily-upgrade.timer
sudo systemctl mask apt-daily.service apt-daily-upgrade.service
```

To confirm:

```bash
systemctl list-timers | grep apt
```

You should see no scheduled APT tasks.

---

## 🧹 Step 3 – Remove Daily Cleanup Scripts (If Created)

If you created the cleanup script in `/usr/local/bin/auto-cleanup.sh`, remove it:

```bash
sudo rm -f /usr/local/bin/auto-cleanup.sh
```

Then edit the cron jobs to remove any automatic cleanup entries:

```bash
sudo crontab -e
```

Look for and delete this line (or similar):

```
30 3 * * * /usr/local/bin/auto-cleanup.sh
```

Save and exit.

---

## 🔁 Step 4 – Optional: Disable Weekly Firmware Update (if enabled)

If you used the “--enable-rpi-firmware” flag in the one-click script, it added a weekly firmware update to root’s crontab.  
To remove it:

```bash
sudo crontab -e
```

Delete the line that looks like this:

```
0 3 * * 0 /usr/bin/rpi-update && /usr/bin/apt autoremove -y && /usr/bin/apt autoclean -y && /bin/systemctl reboot
```

Save and exit.

---

## 🧩 Step 5 – Remove Unattended-Upgrade Configuration Files (Optional)

If you want a completely clean rollback, you can remove or rename the configuration files.

```bash
sudo mv /etc/apt/apt.conf.d/20auto-upgrades /etc/apt/apt.conf.d/20auto-upgrades.disabled
sudo mv /etc/apt/apt.conf.d/50unattended-upgrades /etc/apt/apt.conf.d/50unattended-upgrades.disabled
```

Or simply delete them:

```bash
sudo rm -f /etc/apt/apt.conf.d/20auto-upgrades /etc/apt/apt.conf.d/50unattended-upgrades
```

---

## 🧠 Step 6 – One-Click Disable Script (Optional)

If you prefer an automatic way to disable everything, create this one-click disable script:

```bash
sudo nano disable-auto-updates.sh
```

Paste the following:

```bash
#!/usr/bin/env bash
set -euo pipefail
echo "🛑 Disabling all automatic updates, cleanups, and reboots…"

systemctl stop unattended-upgrades || true
systemctl disable unattended-upgrades || true
systemctl mask unattended-upgrades || true

systemctl disable --now apt-daily.timer apt-daily-upgrade.timer || true
systemctl mask apt-daily.service apt-daily-upgrade.service || true

rm -f /usr/local/bin/auto-cleanup.sh || true

crontab -l 2>/dev/null | grep -v '/usr/local/bin/auto-cleanup.sh' | grep -v 'rpi-update' | crontab - || true

if [ -f /etc/apt/apt.conf.d/20auto-upgrades ]; then
  mv /etc/apt/apt.conf.d/20auto-upgrades /etc/apt/apt.conf.d/20auto-upgrades.disabled
fi
if [ -f /etc/apt/apt.conf.d/50unattended-upgrades ]; then
  mv /etc/apt/apt.conf.d/50unattended-upgrades /etc/apt/apt.conf.d/50unattended-upgrades.disabled
fi

echo
echo "✅ All automatic updates, timers, and scripts have been disabled."
echo "ℹ️ You can re-enable them anytime using your setup-auto-updates.sh script."
```

Make it executable and run it:

```bash
sudo chmod +x disable-auto-updates.sh
sudo ./disable-auto-updates.sh
```

---

## 🔍 Step 7 – Verify That Automation Is Disabled

Check systemd timers and services:

```bash
systemctl list-timers | grep apt
systemctl status unattended-upgrades
```

If everything was successful, no APT timers or unattended-upgrades services will appear active.

---

## ✅ Done!

Your Raspberry Pi 5 is now back to **manual control mode**:

- ❌ Automatic updates disabled  
- ❌ Automatic cleanup disabled  
- ❌ Automatic reboots disabled  
- ⚙️ All configurations saved for easy re-enable later  

You can now manually update your system only when you choose:

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y
```

> 💡 **Tip:** You can re-enable automation anytime using your `auto-update-setup.sh` script to restore full unattended maintenance.
