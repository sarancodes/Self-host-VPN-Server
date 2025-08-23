# Self-host-VPN-Server
A self hosted version of a VPN server with Adguard DNS support.

📖 Overview
-----------

The setup process for running a private VPN and DNS ad-blocking server using:

*   **Amazon EC2** (t3.micro instance)
    
*   **Docker** and **Docker Compose**
    
*   **WireGuard** (VPN server)
    
*   **AdGuard Home** (DNS server with ad blocking)

The main idea is to utilize docker inside your AWS instance to create wireguard and Adguard services to which your clients interact.

## 🛠 Installation
1️⃣ Update System & Install Docker
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose
sudo apt install docker-compose
sudo apt install -y qrencode
```
2️⃣ Create Project Structure
```
mkdir -p ~/server/{wireguard/config,adguard/config,adguard/work}
cd ~/server
```
3️⃣ Create docker-compose.yml
```

cat <<'EOF' > docker-compose.yml
version: "3.8"

services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
      - SERVERURL=<YOUR_INSTANCE_PUBLIC_IP>
      - SERVERPORT=51820
      - PEERS=1
      - PEERDNS=172.20.0.2
    volumes:
      - ./wireguard/config:/config
      - /lib/modules:/lib/modules
    ports:
      - "51820:51820/udp"
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
    networks:
      vpn:
        ipv4_address: 172.20.0.3

  adguard:
    image: adguard/adguardhome
    container_name: adguard
    volumes:
      - ./adguard/config:/opt/adguardhome/conf
      - ./adguard/work:/opt/adguardhome/work
    ports:
      - "80:80"
      - "443:443"
      - "3000:3000"
    restart: unless-stopped
    networks:
      vpn:
        ipv4_address: 172.20.0.2

networks:
  vpn:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24
EOF

```
4️⃣ Start Containers

```
sudo docker-compose up -d
```
## 🔑 Access WireGuard Configuration

**View Client Config:**
```
cat ~/server/wireguard/config/peer1/peer1.conf
```
**Generate QR Code (for mobile clients):**
```
qrencode -t ansiutf8 < ~/server/wireguard/config/peer1/peer1.conf
```
Now install wireguard app into your smartphone and then click on Add button and scan the QR to establish the connection.

**Restart WireGuard:**
```
sudo docker restart wireguard
```

## Setup AdGuard UI Dashboard

Go to 
```
http://<YOUR_INSTANCE_PUBLIC_IP>:3000
```

Then leave everything as defaults
Create user name and password and login

That’s it VPN with Adguard is up and running

## 🌐 AWS Security Group Ports to Open
| Service     | Protocol | Port  |
|-------------|----------|-------|
| SSH         | TCP      | 22    |
| HTTP        | TCP      | 80    |
| HTTPS       | TCP      | 443   |
| AdGuard UI  | TCP      | 3000  |
| WireGuard   | UDP      | 51820 |



