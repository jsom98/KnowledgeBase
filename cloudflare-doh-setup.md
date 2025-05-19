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
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo mv -f ./cloudflared-linux-arm64 /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

Test it works:

```
cloudflared --version
```

---

## 🛠️ Step 3: Create the Cloudflared Service

Create a cloudflared user to run the daemon:

```
sudo useradd -s /usr/sbin/nologin -r -M cloudflared
```

Proceed to create a configuration file for cloudflared:

```
sudo nano /etc/default/cloudflared
```

Edit configuration file by copying the following in to /etc/default/cloudflared. This file contains the command-line options that get passed to cloudflared on startup:

```
# Commandline args for cloudflared, using Cloudflare DNS
CLOUDFLARED_OPTS=--port 5053 --upstream https://cloudflare-dns.com/dns-query
```

You will edit the command line up to the first upstream URLS
Example:

```
# Commandline args for cloudflared, using Cloudflare DNS
CLOUDFLARED_OPTS=--port 5053 --upstream https://2ziabcpo86.cloudflare-gateway.com/dns-query
```

Update the permissions for the configuration file and cloudflared binary to allow access for the cloudflared user:

```
sudo chown cloudflared:cloudflared /etc/default/cloudflared
sudo chown cloudflared:cloudflared /usr/local/bin/cloudflared
```

Then create the systemd script by copying the following into /etc/systemd/system/cloudflared.service. This will control the running of the service and allow it to run on startup:

```
sudo nano /etc/systemd/system/cloudflared.service
```

```
[Unit]
Description=cloudflared DNS over HTTPS proxy
After=syslog.target network-online.target

[Service]
Type=simple
User=cloudflared
EnvironmentFile=/etc/default/cloudflared
ExecStart=/usr/local/bin/cloudflared proxy-dns $CLOUDFLARED_OPTS
Restart=on-failure
RestartSec=10
KillMode=process

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
