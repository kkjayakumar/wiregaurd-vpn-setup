# WireGuard VPN Setup Guide

This guide provides step-by-step instructions to install and configure WireGuard VPN on Ubuntu 20.04 LTS (or newer), including example server and client configurations.

---

## Prerequisites

- Linux Server running **Ubuntu 20.04 LTS** or newer  
  (For other distributions, follow the [official WireGuard installation instructions](https://www.wireguard.com/install/).)

---

## Installation

Install WireGuard on both the **Server** and **Client**:

```sh
sudo apt update
sudo apt install wireguard
```

---

## Key Generation

Generate private and public keys on **both** Server and Client:

```sh
wg genkey | tee privatekey | wg pubkey > publickey
```

> **Note:**  
> **Never share your private key!** Store it securely on each device.

---

## Server Configuration

Create `/etc/wireguard/wg0.conf` on the **server**:

```ini
[Interface]
PrivateKey = <server-private-key>
Address = <server-ip-address>/<subnet>
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o <public-interface> -j MASQUERADE;
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o <public-interface> -j MASQUERADE;
ListenPort = 51820
```

- Replace `<server-private-key>` with your generated private key.
- Set `<server-ip-address>` to a unique private IP (e.g., `10.2.0.72`).
- Set `<subnet>` (e.g., `24`).
- Replace `<public-interface>` with your network interface (e.g., `ens5`, `eth0`).  
  Find it using `ip route show`.

### Example

```ini
[Interface]
Address = 10.2.0.72/24
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE;
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens5 -j MASQUERADE;
ListenPort = 51820
PrivateKey = SHjRYwNYeXEb4I/wKwmmY9ewZcamNVYzU2uotjRFI0k=

[Peer]
PublicKey = C6HodTNIjSI11ga1XYR21ayXl/XGsaijofNw0oP/n00=
AllowedIPs = 10.2.0.89/32
```
Enable the interface on both server and client:

```sh
chmod 777 .
sudo wg-quick up wg0
sudo wg show

```

```sh
sysctl net.ipv4.ip_forward
```
If the output is net.ipv4.ip_forward = 0, it is disabled. You need it to be 1.
```sh
sudo sysctl -w net.ipv4.ip_forward=1
```
Now, try ping 8.8.8.8 from your client device again. If it works, this was the issue!
```sh
```
This rule "masquerades" traffic from your VPN clients, making it look like it originated from your server's public IP.
```sh
sudo iptables -t nat -L POSTROUTING -v -n
```
example
```sh
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   ...
    0     0 MASQUERADE  all  --  *      ens5    10.0.229.0/24        0.0.0.0/0

```
```sh
sudo iptables -L FORWARD -v -n
```
example
```sh
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   ...
   84  7056 ACCEPT     all  --  wg0    *       0.0.0.0/0            0.0.0.0/0

```
```sh
sudo wg show

```

---

## Client Configuration

Create `/etc/wireguard/wg0.conf` on the **client**:

```ini
[Interface]
PrivateKey = <client-private-key>
Address = <client-ip-address>/<subnet>
DNS = 8.8.8.8

[Peer]
PublicKey = <server-public-key>
Endpoint = <server-public-ip-address>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

- Replace `<client-private-key>` with your generated private key.
- Set `<client-ip-address>` to a unique IP in the same subnet as the server (e.g., `10.2.0.89`).
- Replace `<server-public-key>` with the server's public key.
- Replace `<server-public-ip-address>` with the server's public IP.
- aws dns  `10.2.0.2`

### Example

```ini
[Interface]
PrivateKey = WGeD5jefBgMrQspb4fTgfjMmITc30Uhj5l3OCcBz21Y=
Address = 10.2.0.89/32
DNS = 10.2.0.2, 1.1.1.1

[Peer]
PublicKey = f/v0kgEJ88TtgIjZzcryh/RZmoCR3FVN5BHlk2b/f3U=
AllowedIPs = 0.0.0.0/1, 128.0.0.0/1
Endpoint = 13.201.36.154:51820
PersistentKeepalive = 25

```


> **Note:**  
> Setting `AllowedIPs = 0.0.0.0/0` routes **all** client traffic through the VPN.  
> To only route specific networks, adjust this value.

---

## Adding Client to Server

Add the client's public key and IP to the server:

```sh
wg set wg0 peer <client-public-key> allowed-ips <client-ip-address>/32
```

### Example
```sh
sudo wg set wg0 peer HByVG0IIOLHO5qfEa8fyraMcthBtGcWX7p9WjfO+UhI= allowed-ips 10.2.0.90/32
```
---

## Starting WireGuard

Enable the interface on both server and client:

```sh
sudo wg-quick save wg0
sudo wg-quick down wg0
sudo wg-quick up wg0
```

Check status:

```sh
wg
```

---

TEST:

```sh
ping 8.8.8.8
```
if it working is fine
---
## Troubleshooting

- Use `ip route show` to determine your server's default network interface (e.g., `ens5`, `eth0`).
Check status:

```sh
ip route show

```
- Ensure `PostUp` and `PostDown` rules use the correct interface.
- If you can only access the server but not the internet via the tunnel, double-check the interface name in your `MASQUERADE` rules.

---

## References

- [WireGuard Official Documentation](https://www.wireguard.com/)
- [WireGuard Quick Start](https://www.wireguard.com/quickstart/)
