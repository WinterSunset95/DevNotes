# 🧠 Building a Wi-Fi Access Point with hostapd + dnsmasq on Linux

## 📜 Table of Contents
- [1. Overview](#1-overview)
- [2. Requirements](#2-requirements)
- [3. Step-by-Step Guide](#3-step-by-step-guide)
  - [3.1. Check Wireless Card Capabilities](#31-check-wireless-card-capabilities)
  - [3.2. Create a Virtual Interface in AP Mode](#32-create-a-virtual-interface-in-ap-mode)
  - [3.3. Configure hostapd](#33-configure-hostapd)
  - [3.4. Troubleshooting hostapd Errors](#34-troubleshooting-hostapd-errors)
  - [3.5. Assign Static IP to AP Interface](#35-assign-static-ip-to-ap-interface)
  - [3.6. Configure dnsmasq for DHCP + DNS](#36-configure-dnsmasq-for-dhcp--dns)
  - [3.7. Enable IP Forwarding + NAT (Optional)](#37-enable-ip-forwarding--nat-optional)
  - [3.8. Final Testing](#38-final-testing)
- [4. Common Pitfalls](#4-common-pitfalls)
- [5. Optional Improvements](#5-optional-improvements)
- [6. Conclusion](#6-conclusion)

---

## 1. Overview

This guide walks you through creating a Wi-Fi Access Point on Linux using `hostapd` for broadcasting and `dnsmasq` for DHCP/DNS. We’ll also look at NAT configuration to share internet from a second wireless interface.

---

## 2. Requirements

- Linux machine with a Wi-Fi card that supports **AP mode**
- `hostapd`
- `dnsmasq`
- Two interfaces:
  - `myAp` — virtual or physical Wi-Fi interface in AP mode
  - `wlan0` — internet-connected interface

---

## 3. Step-by-Step Guide

---

### 3.1 Check Wireless Card Capabilities

Use this to check if AP mode is supported:

```bash
iw list | grep -A 10 'Supported interface modes'
```

You should see:

```
Supported interface modes:
    * IBSS
    * managed
    * AP
```

---

### 3.2 Create a Virtual Interface in AP Mode

If your card supports simultaneous managed + AP mode on the same channel:

```bash
sudo iw dev wlan0 interface add myAp type __ap
```

To delete it:

```bash
sudo iw dev myAp del
```

---

### 3.3 Configure hostapd

Create `/etc/hostapd/hostapd.conf`:

```ini
interface=myAp
ssid=NewTestAp
hw_mode=a
channel=1
driver=nl80211
```

Start `hostapd` manually for testing:

```bash
sudo hostapd /etc/hostapd/hostapd.conf
```

---

### 3.4 Troubleshooting hostapd Errors

| Error | Fix |
|------|-----|
| `Failed to set beacon parameters` | Interface might be down or not in AP mode |
| `Could not set interface flags (UP): Name not unique` | Interface name may conflict, try a different one |
| `Could not determine operating frequency` | Channel mismatch or interface not ready |
| `Segmentation fault` | Kernel/driver bug or conflicting processes (e.g., NetworkManager) |

Make sure interface is up:

```bash
sudo ip link set myAp up
```

---

### 3.5 Assign Static IP to AP Interface

```bash
sudo ip addr add 192.168.10.1/24 dev myAp
sudo ip link set myAp up
```

---

### 3.6 Configure dnsmasq for DHCP + DNS

Create `/etc/dnsmasq.myap.conf`:

```ini
interface=myAp
bind-interfaces
port=5353

dhcp-range=192.168.10.100,192.168.10.200,255.255.255.0,24h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1

# Optional captive portal behavior
address=/#/192.168.10.1
```

Run it:

```bash
sudo dnsmasq --conf-file=/etc/dnsmasq.myap.conf
```

---

### 3.7 Enable IP Forwarding + NAT (Optional)

To allow clients to access internet from `wlan0`:

```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Set up NAT
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i myAp -o wlan0 -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o myAp -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Make it persistent later using `iptables-save` or a systemd unit.

---

### 3.8 Final Testing

From a client device:

- Connect to SSID `NewTestAp`
- Check IP: should be `192.168.10.x`
- Check gateway/DNS: should be `192.168.10.1`
- Test ping to host machine and 8.8.8.8
- Try browsing the web if NAT is configured

---

## 4. Common Pitfalls

| Problem | Explanation |
|--------|-------------|
| `Segmentation fault in hostapd` | Usually means bad driver or missing permissions |
| `myAp does not come up` | Check that you created the interface properly |
| Clients connect but no internet | IP forwarding or NAT rules missing |
| IP assigned but no DNS | dnsmasq not responding or port conflict |
| Can't create AP while connected to Wi-Fi | Your card might not support simultaneous AP + managed mode |

---

## 5. Optional Improvements

- Use `iptables-persistent` or a custom systemd service for iptables
- Host a captive portal using `nginx` + `nodogsplash` or `php`
- Replace `dnsmasq` with `ISC DHCPD` and `unbound` for finer control
- Log traffic with `tcpdump -i myAp`
- Intercept traffic with `mitmproxy` or `sslstrip` (lab/testing only)

---

## 6. Conclusion

You’ve now built a basic but flexible wireless access point using `hostapd` and `dnsmasq`. This gives you full control over the clients that connect, the DHCP leases they get, and the traffic they send. From here, you can build firewalls, captive portals, fake DNS responses, or traffic loggers for research and testing.
