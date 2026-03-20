Here is the English translation of the **two previous sections** (Docker WireGuard guide + Cloudflare API key/token setup):

---

# 📦 Full Guide: Cloudflare DDNS + WireGuard (Docker)

## 1. Prepare project directory

```bash
mkdir -p ~/vpn-server
cd ~/vpn-server
```

---

## 2. Create `docker-compose.yml`

```yaml
version: "3"

services:
  cloudflare-ddns:
    image: oznu/cloudflare-ddns
    container_name: cf-ddns
    environment:
      - API_TOKEN=your_api_token
      - ZONE=yourdomain.com
      - SUBDOMAIN=vpn
      - PROXIED=false
    restart: always

  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - SERVERURL=vpn.yourdomain.com
      - SERVERPORT=51820
      - PEERS=3
      - PEERDNS=8.8.8.8
      - INTERNAL_SUBNET=10.13.13.0
    volumes:
      - ./wg-config:/config
      - /lib/modules:/lib/modules
    ports:
      - "51820:51820/udp"
    restart: unless-stopped
```

---

## 3. DNS Setup on Cloudflare

* Create record:

  ```
  vpn.yourdomain.com
  ```
* Type: A
* Proxy: **DNS only (must be OFF)**

---

## 4. Open firewall

```bash
sudo ufw allow 51820/udp
```

If behind a router:
→ Forward UDP port **51820** to your server

---

## 5. Start services

```bash
docker compose up -d
```

---

## 6. Generated config

```bash
./wg-config/
```

Contains:

```
peer1/peer1.conf
peer2/
peer3/
wg0.conf
```

---

## 7. Get client config

```bash
cat wg-config/peer1/peer1.conf
```

Or:

```bash
docker exec -it wireguard /app/show-peer 1
```

---

## 8. Client config example

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.13.13.2
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = vpn.yourdomain.com:51820
AllowedIPs = 0.0.0.0/0
```

---

## 9. Connect client

Install WireGuard on:

* Windows
* Android
* iOS

Import config file

---

## 10. Test

```bash
curl ifconfig.me
```

→ Should return server IP

---

## 11. Add peer

```bash
docker exec -it wireguard /app/add-peer newuser
```

Show:

```bash
docker exec -it wireguard /app/show-peer newuser
```

---

## 12. Remove peer

```bash
docker exec -it wireguard /app/remove-peer newuser
```

---

# 🔐 Cloudflare API Key / Token Guide

## 1. Recommended: Use API Token

Go to:
[https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)

---

## 2. Create token

* Click **Create Token**
* Choose template: `Edit zone DNS`

---

## 3. Permissions

```
Zone → DNS → Edit
Zone → Zone → Read
```

---

## 4. Zone Resources

```
Include → Specific zone → yourdomain.com
```

---

## 5. Use in Docker

```yaml
environment:
  - API_TOKEN=your_api_token
  - ZONE=yourdomain.com
  - SUBDOMAIN=vpn
  - PROXIED=false
```

---

## 6. Restart

```bash
docker compose down
docker compose up -d
```

---

## 7. Verify logs

```bash
docker logs cf-ddns
```

Expected:

```
Updated DNS record to x.x.x.x
```

---

## 8. Common issues

| Issue            | Cause                     |
| ---------------- | ------------------------- |
| 403 error        | wrong token / permissions |
| DNS not updating | wrong zone/subdomain      |
| VPN not working  | Cloudflare proxy enabled  |

---

# ✅ Final checklist

* DNS record exists
* Proxy OFF
* API token valid
* Port 51820 open
* Containers running

---

If you want, I can also provide:

* Production-ready `.env`
* Secure setup (fail2ban, firewall rules)
* WireGuard UI (wg-easy)

Just tell me 👍
