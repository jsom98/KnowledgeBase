# 🧪 Raspberry Pi 5 – Pi-hole Setup & Web Portal Access

---

**Last updated:** [5/19/2025]  
**Author:** Jeffrey Som  
**Tags:** Raspberry Pi, Pi-hole, DNS, Ad-blocking, Network Security

---

## 📝 Overview

Installing **Pi-hole** on your Raspberry Pi transforms it into a network-wide ad blocker — no browser extensions needed. It acts as a local DNS server, intercepting and dropping unwanted tracking domains before they even reach your devices.

In this guide, you’ll learn how to:

- Install Pi-hole using the official script
- Access the web dashboard
- Set (or reset) the password for portal login
- Verify that Pi-hole is working

Let’s get started 🚀

---


## 🚀 What You’ll Need

✅ A Raspberry Pi 5  
✅ Raspberry Pi OS (Lite or Desktop)  
✅ Terminal access (keyboard, SSH, or remote tools)  
✅ Your desired static IP info (IP, gateway, DNS)

---

## ⚙️ Step 1: Update Your Pi

Always begin by refreshing your Raspberry Pi’s system:

```
sudo apt update && sudo apt full-upgrade -y
```

---

## 📦 Step 2: Install Pi-hole

Run the official one-liner script:

```
curl -sSL https://install.pi-hole.net | bash
```

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS9.png)

This will launch a text-based installer. Walk through the prompts carefully. Choose your network interface, upstream DNS provider (Cloudflare is a solid pick), and whether to block IPv6.

---

## 🌐 Step 3: Access the Admin Dashboard

After setup, the Pi-hole dashboard is available here:

```
http://<your-pi-ip>/admin
```

Example:
```
http://192.168.5.10/admin
```

Bookmark this page — it’s where you manage blacklists, view DNS logs, and see how many ads are blocked!

---

## 🔐 Step 4: Set or Change the Web Portal Password

Use the following command to secure your dashboard:

```
pihole setpassword
```

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS10.png)

🔸 Leave the password blank to remove it (not recommended).  
🔸 Run this again anytime you need to reset access.

---

## 🧪 Step 5: Test It Out

Set another device on your network to use your Pi-hole's IP as its DNS server:

```
DNS: 192.168.5.10
```

Then browse the web normally — ads should disappear!

You can also view logs live with:

```
pihole -t
```

---

## 🧼 Maintenance & Extras

- Update Pi-hole:
  ```
  pihole -up
  ```

- Refresh blocklists:
  ```
  pihole -g
  ```

- Temporarily disable:
  ```
  pihole disable
  ```

- Re-enable:
  ```
  pihole enable
  ```

---

## ✅ That’s It!

Your network just leveled up in privacy and speed.  
With Pi-hole running, you're free from ads and background tracking — all from a $35 Raspberry Pi. 🙌

Let me know if you want to take it further with:
- 🚪 PiVPN integration
- 🔒 Unbound for DNS over HTTPS
- ☁️ Cloud backups
