# 📡 Raspberry Pi 5 – Static IP Configuration Guide

> Last updated: [Insert Date]  
> Author: Jeffrey Som  
> Tags: Raspberry Pi, Static IP, Networking, DHCP, Linux

---

## 📝 Overview

Assigning a **static IP address** to your Raspberry Pi ensures that its network identity stays consistent across reboots and power cycles—especially helpful for headless setups or when hosting local services like Pi-hole.

This guide walks through configuring a **static IP address** on a Raspberry Pi 5 running Raspberry Pi OS (Bookworm or Bullseye), using both CLI (headless) and GUI (desktop) methods.

---

## 🧰 Prerequisites

- Raspberry Pi 5 (or 4/3) with Raspberry Pi OS installed  
- Access via:  
  - SSH _(for headless setups)_  
  - Desktop interface _(if GUI is enabled)_  
- Network info:
  - Example Static IP: `192.168.5.10`
  - Gateway/Router IP: `192.168.5.1`
  - DNS servers: `1.1.1.1`, `8.8.8.8`

---

## 🖥️ CLI Method (Recommended for Headless)

### ✅ 1. Update and Upgrade System

```bash
sudo apt update && sudo apt full-upgrade -y
