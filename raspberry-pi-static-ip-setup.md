# 🧠 Raspberry Pi 5 – Static IP Address Setup (KB Guide)

**Goal:** Set up a static IP address on your Raspberry Pi 5 so you can always find it on your network — no more guessing which IP it grabbed this time!

---

## 🚀 What You’ll Need

✅ A Raspberry Pi 5  
✅ Raspberry Pi OS (Lite or Desktop)  
✅ Terminal access (keyboard, SSH, or remote tools)  
✅ Your desired static IP info (IP, gateway, DNS)

---

## ⚙️ Step 1: Update Everything First!

Always begin by making sure your system is current:

```
sudo apt update && sudo apt full-upgrade -y
```

This helps avoid weird surprises later.

---

## 📦 Step 2: Make Sure `dhcpcd` is Installed

Just in case it’s missing or corrupted, reinstall it:

```
sudo apt install --reinstall dhcpcd5 -y
```

Think of `dhcpcd` as your Pi's network butler—it needs to be around to set the rules.

---

## 📁 Step 3: Backup Your DHCP Config (Smart Move)

Before we change anything, let’s make a backup:

```
sudo cp /etc/dhcpcd.conf /etc/dhcpcd.conf.backup
```

If things go sideways, you’ll thank yourself later.

---

## 🛠️ Step 4: Set the Static IP

Open the config file:

```
sudo nano /etc/dhcpcd.conf
```

Now scroll to the bottom and add your settings. Example for Ethernet (`eth0`):

```
interface eth0
static ip_address=192.168.5.10/24
static routers=192.168.5.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

✨ *If you're on Wi-Fi, change `eth0` to `wlan0`.*

🧠 **Tip:** Make sure the static IP you choose is *outside* your router’s DHCP pool to avoid conflicts.

---

## 🔄 Step 5: Restart the Network Service

Let the changes take effect:

```
sudo systemctl restart dhcpcd
```

Or just reboot if you want to be thorough:

```
sudo reboot
```

---

## 🔍 Step 6: Confirm It Worked

Once you're back up:

```
ip a
```

Check that your IP under `eth0` (or `wlan0`) matches what you set.

---

## ✅ Done & Connected!

Boom 💥 — your Pi now has a consistent home on your network. No more playing "Find That Pi" with your router.

---

## 🧠 Bonus Tips

- Use `nmap -sn 192.168.5.0/24` from another device to scan your local network.
- Label your Pi’s MAC address in your router’s settings for extra clarity.
- Want to do this through the desktop UI? Check your “Network Preferences” under IPv4 settings.

---

Let me know if you want a PDF export, GitHub README, or version for Pi-hole setups next!
