# ☁️ Raspberry Pi 5 – Cloudflare DNS-over-HTTPS (DoH) Setup

---

**Last updated:** [5/19/2025]  
**Author:** Jeffrey Som  
**Tags:** Raspberry Pi, Cloudflare, DNS-over-HTTPS, DoH, Privacy, Network Security

---

## 📝 Overview

Want to keep your DNS queries secure and private? Setting up **Cloudflare DNS-over-HTTPS (DoH)** ensures your DNS requests are encrypted — even from your ISP. This guide walks through installing **Cloudflared**, a lightweight proxy that enables encrypted DNS resolution with Pi-hole on your Raspberry Pi 5.

---

## 🚀 What You’ll Need

✅ Raspberry Pi 5 running Raspberry Pi OS  
✅ Pi-hole installed and working  
✅ Internet access  
✅ Terminal access (SSH or direct)

---

## ⚙️ Step 1: Update Your System

```
sudo apt update && sudo apt full-upgrade -y
```

---

## 📥 Step 2: Install Cloudflared

```
sudo apt install curl -y
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64 -o cloudflared
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin
```

Test it works:

```
cloudflared --version
```

---

## 🛠️ Step 3: Create the Cloudflared Service

```
sudo nano /etc/systemd/system/cloudflared.service
```

Paste this:

```
[Unit]
Description=cloudflared DNS over HTTPS proxy
After=network.target

[Service]
ExecStart=/usr/local/bin/cloudflared proxy-dns --port 5053 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query
Restart=on-failure
User=nobody

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Check its status:

```
sudo systemctl status cloudflared
```

---

## ⚙️ Step 4: Configure Pi-hole to Use DoH

Go to:

**Pi-hole Admin Panel → Settings → DNS**  
Uncheck all upstream DNS providers.  
Under "Custom 1 (IPv4)", enter:

```
127.0.0.1#5053
```

Click "Save".

---

## 🔎 Step 5: Confirm DNS-over-HTTPS Is Working

Run:

```
dig @127.0.0.1 -p 5053 cloudflare.com
```

You should get a valid response. That means Cloudflared is encrypting your DNS traffic.

---

## ✅ Done!

Your Raspberry Pi is now using **Cloudflare DNS-over-HTTPS**, making your DNS lookups private, encrypted, and tamper-resistant.

Let me know if you'd like a diagram comparing DNS, DNS-over-TLS, and DoH — or how to layer this with Unbound for even more control!
