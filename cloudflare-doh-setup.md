# ☁️ Raspberry Pi 5 – Cloudflare DNS-over-HTTPS (DoH) Setup

**Last updated:** 5/19/2025


**Author:** Jeffrey Som


**Tags:** Raspberry Pi, Cloudflare, DNS-over-HTTPS, DoH, Privacy, Network Security



## 📝 Overview

Want to keep your DNS queries secure and private? Setting up Cloudflare DNS-over-HTTPS (DoH) ensures your DNS requests are encrypted, even from your ISP.

**Why this matters:** Every time you visit a website, your device first asks a DNS server, "What's the IP address for this name?" Normally that question is sent in plain text, so your ISP (or anyone in between) can see every site you look up. Pi-hole blocks ads, but it still sends its lookups out in plain text. DoH wraps those lookups in HTTPS, the same encryption websites use, so they can't be read or tampered with.

This guide walks through installing **Cloudflared**, a lightweight proxy that sits between Pi-hole and Cloudflare. Pi-hole hands its DNS queries to Cloudflared, and Cloudflared encrypts them and sends them to Cloudflare.

```
Your devices → Pi-hole → Cloudflared (on the Pi) → Cloudflare (encrypted)
```

## 🚀 What You'll Need

- ✅ Raspberry Pi 5 running Raspberry Pi OS
- ✅ Pi-hole installed and working
- ✅ Internet access
- ✅ Terminal access (SSH or direct)

## ⚙️ Step 1: Update Your System

```bash
sudo apt update && sudo apt full-upgrade -y
```

**What this does:** `apt update` refreshes the list of available software, and `full-upgrade` installs the latest versions. Starting from an up-to-date system helps avoid errors and gives you the latest security fixes. The `-y` answers "yes" automatically so it doesn't stop to ask.

## 📥 Step 2: Install Cloudflared

> **Note:** Cloudflare removed the `proxy-dns` feature (used in Step 3) starting with cloudflared version **2026.2.0**. The `latest` download link below gets a version without it. To use this guide, download an older release from the [cloudflared releases page](https://github.com/cloudflare/cloudflared/releases) by replacing `<version>` with a release number earlier than 2026.2.0:
> `wget https://github.com/cloudflare/cloudflared/releases/download/<version>/cloudflared-linux-arm64`

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo mv -f ./cloudflared-linux-arm64 /usr/local/bin/cloudflared
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

**What each line does:**
- `wget ...` downloads the Cloudflared program. `arm64` is the version built for the Raspberry Pi 5's 64-bit processor.
- `sudo mv ...` moves it to `/usr/local/bin` and renames it to `cloudflared`. Programs in that folder can be run from anywhere just by typing their name.
- `sudo chmod +x ...` marks the file as executable. Downloaded files can't be run as programs until you do this.
- `cloudflared -v` prints the version number. If you see a version, the install worked.

## 🛠️ Step 3: Create the Cloudflared Service

Create a `cloudflared` user to run the daemon:

```bash
sudo useradd -s /usr/sbin/nologin -r -M cloudflared
```

**Why a separate user:** Running Cloudflared under its own limited account means that if something ever went wrong with the program, it wouldn't have full control of your Pi.
- `-s /usr/sbin/nologin` means nobody can log in as this user.
- `-r` makes it a "system" account, used only for running services.
- `-M` skips creating a home folder, since it doesn't need one.

Next, create a configuration file for cloudflared:

```bash
sudo nano /etc/default/cloudflared
```

Copy the following into `/etc/default/cloudflared`. This file holds the command-line options that get passed to cloudflared when it starts:

```
# Commandline args for cloudflared, using Cloudflare DNS
CLOUDFLARED_OPTS=--port 5053 --upstream https://cloudflare-dns.com/dns-query
```

**What these options mean:**
- `--port 5053` tells Cloudflared to listen on port 5053. Pi-hole already uses the standard DNS port, 53, so Cloudflared needs a different one.
- `--upstream https://cloudflare-dns.com/dns-query` is Cloudflare's DoH address, where the encrypted queries get sent.

Save and exit nano with `Ctrl+O`, `Enter`, then `Ctrl+X`.

**Optional:** If you use Cloudflare Zero Trust (Gateway), replace the upstream URL with your own Gateway DoH address. This lets you use your own Cloudflare filtering rules and logs. Example:

```
# Commandline args for cloudflared, using Cloudflare DNS
CLOUDFLARED_OPTS=--port 5053 --upstream https://2ziabcpo86.cloudflare-gateway.com/dns-query
```

Update the permissions for the configuration file and cloudflared binary so the cloudflared user can access them:

```bash
sudo chown cloudflared:cloudflared /etc/default/cloudflared
sudo chown cloudflared:cloudflared /usr/local/bin/cloudflared
```

**What this does:** `chown` changes who owns a file. Here it makes the `cloudflared` user the owner of its config file and program.

Next, create the systemd service file. **systemd** is the part of Raspberry Pi OS that starts and manages background programs. This file tells it how to run Cloudflared and to start it automatically on boot.

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

Copy the following into the file:

```ini
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

**What the important lines mean:**
- `After=... network-online.target` waits for the network to be up before starting, since Cloudflared needs internet to work.
- `User=cloudflared` runs it as the limited user you created earlier.
- `EnvironmentFile=/etc/default/cloudflared` loads your settings from the config file you made.
- `ExecStart=... proxy-dns $CLOUDFLARED_OPTS` is the actual command that runs. `proxy-dns` puts Cloudflared in DNS proxy mode, and `$CLOUDFLARED_OPTS` fills in your port and upstream settings.
- `Restart=on-failure` and `RestartSec=10` automatically restart Cloudflared 10 seconds after it crashes.
- `WantedBy=multi-user.target` starts it automatically at boot.

Enable and start the service:

```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

**What this does:** `enable` makes Cloudflared start automatically every time the Pi boots. `start` runs it right now.

Check its status:

```bash
sudo systemctl status cloudflared
```

You should see **`active (running)`** in green. Press `q` to exit.

## ⚙️ Step 4: Configure Pi-hole to Use DoH

Go to:

**Pi-hole Admin Panel → Settings → DNS**

1. Uncheck all upstream DNS providers.
2. Under **"Custom 1 (IPv4)"**, enter the line below. On newer Pi-hole versions (v6), enter it in the **"Custom DNS servers"** box instead.

```
127.0.0.1#5053
```

3. Click **Save**.

**Why we do this:**
- **Unchecking the other providers:** Pi-hole sends queries to *every* checked server. If any stay checked, some of your lookups would skip Cloudflared and go out unencrypted.
- **`127.0.0.1#5053`:** `127.0.0.1` means "this same Pi," and `#5053` is the port Cloudflared is listening on. This tells Pi-hole to send all of its queries to Cloudflared.

## 🔎 Step 5: Confirm DNS-over-HTTPS Is Working

Run:

```bash
dig @127.0.0.1 -p 5053 cloudflare.com
```

**What this does:** `dig` is a tool for testing DNS. This command asks Cloudflared directly (port 5053 on this Pi) to look up `cloudflare.com`. If `dig` isn't installed, install it with `sudo apt install dnsutils -y`.

You should get a valid response, with `status: NOERROR` and an IP address in the **ANSWER SECTION**. That means Cloudflared is working and encrypting your DNS traffic.

**Extra check:** On any device that uses your Pi-hole, visit [https://1.1.1.1/help](https://1.1.1.1/help). It should say **"Using DNS over HTTPS (DoH): Yes."**

## ✅ Done!

Your Raspberry Pi is now using Cloudflare DNS-over-HTTPS, making your DNS lookups private, encrypted, and tamper-resistant.
