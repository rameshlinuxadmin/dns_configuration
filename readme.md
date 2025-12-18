# BIND DNS Server Setup Guide

This document explains how to install and configure a **BIND (named)** DNS server with **forward** and **reverse** lookup zones on a Linux system.

---

## 📦 Package Information

| Item | Value |
|-----|------|
| Package Name | `bind` |
| Service Name | `named` |
| Port | `53` |
| Root Directory | `/var/named` |
| Main Config File | `/etc/named.conf` |

---

## Step 1: Install BIND Package

```bash
yum install bind -y
rpm -qa | grep -i bind
```

---

## Step 2: Update BIND Configuration

Edit the main configuration file:

```bash
vi /etc/named.conf
```

Allow DNS queries from **any IP address** (or restrict to a subnet if required):

```conf
options {
    listen-on port 53 { 127.0.0.1; any; };
    ...
    ...
    allow-query { localhost; any; };
};
```

> 🔐 **Note:**
> - Use `any` to allow queries from all IPs
> - Use a subnet (example: `10.0.0.0/8`) to restrict access

---

## Forward Lookup Zone Configuration

Example domain: **aravind.com**

```conf
zone "aravind.com." IN {
    type master;                     # Type of DNS server
    file "aravind.com.forward.db";  # Forward lookup DB file
};
```

---

## Reverse Lookup Zone Configuration

Reverse zone depends on the IP class:

- **Class A:** `10.in-addr.arpa`
- **Class B:** `25.172.in-addr.arpa`
- **Class C:** `12.25.192.in-addr.arpa`

Example (Class B):

```conf
zone "25.172.in-addr.arpa" IN {
    type master;                      # Type of DNS server
    file "aravind.com.reverse.db";   # Reverse lookup DB file
};
```

---

## Step 3: Create Forward Lookup DB File

Location: `/var/named/aravind.com.forward.db`

```dns
$TTL 1D
@   IN SOA  master.aravind.com. root.master.aravind.com. (
            3       ; serial
            1D      ; refresh
            1H      ; retry
            1W      ; expire
            3H )    ; minimum

@       IN NS   master.aravind.com.
@       IN NS   slave.aravind.com.
@       IN A    172.25.250.9

master  IN A    172.25.250.9
slave   IN A    172.25.250.10
client  IN A    172.25.250.11
```

---

## Step 4: Create Reverse Lookup DB File

Location: `/var/named/aravind.com.reverse.db`

```dns
$TTL 1D
@   IN SOA  master.aravind.com. root.master.aravind.com. (
            6       ; serial
            1D      ; refresh
            1H      ; retry
            1W      ; expire
            3H )    ; minimum

@   IN NS   master.aravind.com.
@   IN NS   slave.aravind.com.

@   IN A    172.25.250.9
@   IN A    172.25.250.10

9.250   IN PTR  master.aravind.com.
10.250  IN PTR  slave.aravind.com.
11.250  IN PTR  client.aravind.com.
```

---

## ✅ Final Notes

- Ensure correct file ownership and permissions:

```bash
chown root:named /var/named/*.db
chmod 640 /var/named/*.db
```

- Validate configuration:

```bash
named-checkconf
named-checkzone aravind.com /var/named/aravind.com.forward.db
named-checkzone 25.172.in-addr.arpa /var/named/aravind.com.reverse.db
```

- Start and enable DNS service:

```bash
systemctl start named
systemctl enable named
```

---

🎯 **Your BIND DNS server is now ready with forward and reverse lookup support.**

