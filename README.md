# Raspberry Pi Pi-hole + Tailscale Gateway Setup

This guide walks through setting up a Raspberry Pi to function as:

- 🛡 Pi-hole (Network-wide ad blocking)
- 🌐 Tailscale Subnet Router (Access local LAN remotely)
- 🚪 Tailscale Exit Node (Full-tunnel VPN back to home)
- 🔒 Secure remote gateway with no port forwarding required

---

## 🧰 Requirements

- Raspberry Pi (64-bit recommended)
- microSD card
- Raspberry Pi Imager
- Internet connection
- Tailscale account

---

# 1️⃣ Install Ubuntu Server

1. Open **Raspberry Pi Imager**
2. Select:
   - **Device** → Your Raspberry Pi
   - **OS** → Ubuntu Server (64-bit recommended)
   - **Storage** → SD card

3. Open Advanced Options (gear icon):
   - Set hostname (e.g. `pi-gateway`)
   - Enable SSH
   - Set username & password
   - Configure WiFi (if needed)

4. Flash SD card and boot the Pi.

---

# 2️⃣ Update Ubuntu

SSH into the Pi:

```bash
ssh username@pi-ip-address
```

Update packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl -y
```

Reboot if needed:

```bash
sudo reboot
```

---

# 3️⃣ Install Pi-hole

Install using the official script:

```bash
curl -sSL https://install.pi-hole.net | bash
```

During setup:

- Select correct network interface (eth0 or wlan0)
- Assign a static IP (recommended)
- Choose upstream DNS provider
- Enable web admin interface

Set or confirm admin password:

```bash
pihole -a -p
```

Access the dashboard:

```
http://<pi-ip>/admin
```

---

# 4️⃣ Install Tailscale

Install:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate:

```bash
sudo tailscale up
```

Follow the login URL provided.

Verify:

```bash
tailscale status
```

---

# 5️⃣ Enable IP Forwarding (Required for Routing)

Enable immediately:

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf

```

---

# 6️⃣ Advertise Local Subnet

Replace with your actual subnet (example uses 192.168.1.0/24):

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

Then:

1. Go to **Tailscale Admin Console**
2. Select your Raspberry Pi
3. Approve the advertised route

You can now access local devices remotely via their LAN IP.

---

# 7️⃣ Enable Exit Node (Full Tunnel Mode)

Enable exit node:

```bash
sudo tailscale up --advertise-exit-node
```

If an error is shown, run the command with all flags

```
sudo tailscale up --advertise-exit-node --accept-dns=false --accept-routes --advertise-routes=192.168.1.0/24
```

Then:

1. Approve exit node in Tailscale Admin Console
2. On client devices, select the Pi as the Exit Node

All internet traffic will now route through your home network.

---


# 8️⃣ Optional: Use Pi-hole for Tailscale DNS

To enable remote ad blocking:

1. Go to **Tailscale Admin Console**
2. Navigate to **DNS**
3. Set Global Nameserver to your Pi’s Tailscale IP (100.x.x.x)

Remote devices will now use Pi-hole automatically.

---

# 🌐 Network Diagram

## Split Tunnel (Subnet Router Only)

```
            Remote Device
                │
                │ (Tailscale)
                ▼
        ┌─────────────────┐
        │ Raspberry Pi     │
        │ Pi-hole +        │
        │ Tailscale Router │
        └─────────────────┘
                │
                │
        ┌───────────────┐
        │  Home Network │
        │ 192.168.1.0/24│
        └───────────────┘

Remote device internet traffic → Local internet
Only 192.168.1.x traffic → Routed through Pi
```

---

## Full Tunnel (Exit Node Enabled)

```
            Remote Device
                │
                │ (All Traffic via Tailscale)
                ▼
        ┌─────────────────┐
        │ Raspberry Pi     │
        │ Pi-hole +        │
        │ Exit Node        │
        └─────────────────┘
                │
                ▼
          Home Internet
                │
                ▼
             Internet
```

All traffic routes through your home connection.
Remote device behaves as if it is physically at home.

---


# ✅ Final Result

This Raspberry Pi now provides:

- Remote LAN access
- Network-wide ad blocking
- Optional full VPN tunnel
- Secure encrypted connectivity
- No port forwarding required

Your home network is now securely accessible from anywhere.
