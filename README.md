# Setting Up mDNS Reflection Over WireGuard

This guide explains how to set up mDNS reflection between two Ubuntu servers over a WireGuard VPN. This is useful when devices on one network (e.g., IoT devices) need to communicate with a service like Home Assistant on another network using mDNS.

---

## Prerequisites

1. **Two Ubuntu servers**:
   - Server A: Runs Home Assistant (HA).
   - Server B: Hosts IoT devices.
2. **WireGuard installed and configured** on both servers with active connectivity.

---

## Steps

### 1. Install Avahi on Both Servers

Avahi is used to handle mDNS traffic.

```bash
sudo apt update
sudo apt install avahi-daemon avahi-utils
```

Enable the Avahi reflector:

```bash
sudo nano /etc/avahi/avahi-daemon.conf
```

Ensure the following settings are present:

```ini
[reflector]
enable-reflector=yes

[server]
use-ipv4=yes
use-ipv6=yes
```

Restart the Avahi service:

```bash
sudo systemctl restart avahi-daemon
sudo systemctl enable avahi-daemon
```

---

### 2. Configure Multicast Routing

WireGuard does not route multicast traffic by default. To fix this, add a multicast route on both servers.

Add the route for multicast addresses (`224.0.0.0/4`):

```bash
sudo ip route add 224.0.0.0/4 dev wg0
```

Verify the route:

```bash
ip route show
```

You should see:

```
224.0.0.0/4 dev wg0 scope link
```

---

### 3. Update WireGuard Configuration

Ensure WireGuard allows multicast traffic. Edit the WireGuard configuration file on both servers:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add `224.0.0.0/4` to the `AllowedIPs` for each peer:

```ini
[Peer]
PublicKey = <remote-public-key>
AllowedIPs = 10.88.0.0/24, 224.0.0.0/4
```

Save and restart WireGuard:

```bash
sudo systemctl restart [email protected]
```

---

### 4. Test mDNS Reflection

#### **Publish a Test Service on Server B (IoT Server):**

```bash
avahi-publish -s TestService _http._tcp 8080
```

#### **Browse for mDNS Services on Server A (HA Server):**

```bash
avahi-browse -a
```

You should see the `TestService` listed.

---

### 5. Debugging

If mDNS does not work:

1. **Check multicast traffic**:
   
   On the HA server (`wg0`):
   ```bash
   sudo tcpdump -i wg0 port 5353
   ```

2. **Check Avahi logs**:
   ```bash
   sudo journalctl -u avahi-daemon
   ```

3. Ensure the multicast route (`224.0.0.0/4`) is added and `AllowedIPs` includes it in WireGuard.

---

### Summary

With this setup, mDNS traffic should now reflect between the two servers via WireGuard. This allows IoT devices to communicate seamlessly with Home Assistant or other services over a VPN.

---

Feel free to use this guide as a reference or share it with others!
