# 🧠 Pi-hole Setup & Configuration Guide

> Last updated: [5/19/2025]  
> Author: Jeffrey Som  
> Tags: DNS, Raspberry Pi, Ad Blocking, Network Security, Home Lab

---

## 📌 Overview

Pi-hole is a network-wide DNS sinkhole that blocks ads and trackers. It works as a local DNS server and provides real-time analytics for DNS queries on your network.

This guide covers the full setup process for Pi-hole on a Raspberry Pi device, along with optional enhancements and security best practices.

---

## 🧰 Prerequisites

- Raspberry Pi 3, 4, or 5 (Raspberry Pi OS / Debian-based)
- Static IP address configured (e.g. `192.168.5.5`)
- SSH access enabled
- Internet connection
- A client device to test (e.g. phone or laptop)

---

## ⚙️ Step-by-Step Setup

### 1. Update the OS

```bash
sudo apt update && sudo apt upgrade -y
