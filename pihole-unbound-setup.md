# 🔁 Pi-hole + Unbound – Recursive DNS Configuration Guide

---

**Last updated:** [5/19/2025]  
**Author:** Jeffrey Som  
**Tags:** Raspberry Pi, Pi-hole, Unbound, DNS, Privacy, Recursive DNS

---

## 📝 Overview

Pairing **Pi-hole** with **Unbound** transforms your Raspberry Pi into a fully self-reliant DNS server. Instead of relying on third-party DNS providers like Google or Cloudflare, Unbound allows you to **resolve DNS queries directly from the root servers** — boosting privacy and reducing reliance on external resolvers.

This guide covers:

- Installing Unbound on your Pi-hole server
- Setting up the config file for best performance
- Testing your recursive DNS setup

---

## ⚙️ Step 1: Update Your Pi

```
sudo apt update && sudo apt full-upgrade -y
```

---

## 📦 Step 2: Install Unbound

```
sudo apt install unbound -y
```

This installs the Unbound resolver service.

---

## 📁 Step 3: Add the Root Hints File

Unbound needs a list of root DNS servers. Download it:

```
wget -O root.hints https://www.internic.net/domain/named.cache
sudo mv root.hints /var/lib/unbound/root.hints
```

---

## 🛠️ Step 4: Configure Unbound

Create a configuration file for Unbound:

```
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

Paste the following configuration:

```
server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    root-hints: "/var/lib/unbound/root.hints"
    hide-identity: yes
    hide-version: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes
    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1
    so-rcvbuf: 1m
    cache-min-ttl: 3600
    cache-max-ttl: 86400

    private-address: 192.168.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
```

---

## 🔄 Step 5: Restart Unbound

```
sudo service unbound restart
```

---

## ⚙️ Step 6: Point Pi-hole to Unbound

Go to your Pi-hole Admin Panel:

**Settings → DNS → Upstream DNS Servers**  
Uncheck all DNS providers and add this custom one:

```
127.0.0.1#5335
```

Save and reload.

---

## 🧪 Step 7: Test It Out

Run a quick test to verify that Unbound is answering DNS queries:

```
dig pi-hole.net @127.0.0.1 -p 5335
```

You should see a `NOERROR` response near the bottom of the output.

---

## ✅ Done!

Your Raspberry Pi now performs **recursive DNS resolution** on its own, boosting both your network’s **privacy** and **independence**.

You’ve cut out third-party DNS providers and are using the internet’s actual root servers — like a boss 🧠💪

---

Let me know if you'd like a visual flow diagram of this setup or want to take it further with DNS-over-TLS or DoH support!
