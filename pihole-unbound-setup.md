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

## 🚀 What You’ll Need

✅ A Raspberry Pi 5  
✅ Raspberry Pi OS (Lite or Desktop)  
✅ Terminal access (keyboard, SSH, or remote tools)  
✅ Your desired static IP info (IP, gateway, DNS)

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

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS11.png)

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

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS12.png)

Paste the following configuration:

```
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to no if you don't have IPv6 connectivity
    do-ip6: yes

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10

    # Ensure no reverse queries to non-public IP ranges (RFC6303 4.2)
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32
```

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS13.png)

---

## 🔄 Step 5: Restart Unbound

```
sudo service unbound restart
```

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS14.png)

---

## ⚙️ Step 6: Point Pi-hole to Unbound

Go to your Pi-hole Admin Panel:

**Settings → DNS → Upstream DNS Servers**  
Uncheck all DNS providers and add this custom one:

```
127.0.0.1#5335
```

![Untitled](https://github.com/jsom98/KBPictures/blob/main/SS15.png)

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
