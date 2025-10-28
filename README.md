# aztec-docker-guide
Step by step guide to setup Aztec node with Docker  

---

## Hardware Requirements
- **CPU**: 4+ Core  
- **RAM**: 8-16GB  
- **Disk**: 200GB+ SSD  

> 💡 Tip: You can check [servarica.com](https://servarica.com) for affordable servers. Recommended: **KVM Slim Slice 8**.  

---

## 1. Prerequisites  

### Update packages
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Install dependencies
```bash
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip ufw screen gawk -y
```

### Install Docker
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Test Docker
sudo docker run hello-world

sudo systemctl enable docker
sudo systemctl restart docker
```

---

## 2. Gather Required Info 

- **RPC URLs**: Sepolia RPC + Beacon RPC (from third-party provider or self-hosted)  
- **Wallet private key and public address**: Use a freshly generated EVM wallet for security  
- **Server IP address**  

### Find your server IP
```bash
curl ipv4.icanhazip.com
```

Save it somewhere for later use.  

---

## 3. Enable Firewall and Open Ports  

```bash
# Firewall rules
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw enable

# Sequencer ports
sudo ufw allow 40400
sudo ufw allow 8080
sudo ufw reload

# If prompted (y/n), enter 'y'
```
## 4. Prepare Directory

### Delete old container/data (if node was running before)
```bash
# Stop docker container
sudo docker stop $(docker ps -q --filter "ancestor=aztecprotocol/aztec") && sudo docker rm $(docker ps -a -q --filter "ancestor=aztecprotocol/aztec")

# Stop screens
screen -ls | grep -i aztec | awk '{print $1}' | xargs -I {} screen -X -S {} quit

#delete data folder
rm -rf ~/.aztec/alpha-testnet/data/ && rm -rf ~/.aztec/testnet/data/
```
### Create directory
```bash
mkdir aztec
```
### Go into your directory
```bash
cd aztec
```
## 5. create `.ENV` file
```bash
nano .env
```
### paste this and fill out your necessary info
```bash
ETHEREUM_RPC_URL=Sepolia-RPC
CONSENSUS_BEACON_URL=Beacon-RPC
VALIDATOR_PRIVATE_KEYS=0xYourPrivateKey
COINBASE=0xYourAddress
P2P_IP=Your-IP-Address
GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS=0xDCd9DdeAbEF70108cE02576df1eB333c4244C666
```
### Replace the following variables before saving:
- **ETHEREUM_RPC_URL & BEACON_URL** → use `http://localhost:8545` (Sepolia) and `http://localhost:3500` (Beacon) if you are hosting your own RPC and Aztec node on same server
- **VALIDATOR_PRIVATE_KEYS** → Your EVM wallet private key starting with `0x...`  
- **COINBASE** → Your EVM wallet public address starting with `0x...`  
- **P2P_IP** → Your server IPv4
  
  **👉 Save with `CTRL+X`, press `Y`, then Enter.**
  
## 6. Create `docker-compose.yml` file
```bash
nano docker-compose.yml 
```
**Paste this inside the file (dont edit or change anything. just paste as shown)
```yaml
services:
  aztec-node:
    container_name: aztec-sequencer
    network_mode: host
    image: aztecprotocol/aztec:2.0.4
    restart: unless-stopped
    environment:
      GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS: ${GOVERNANCE_PROPOSER_PAYLOAD_ADDRESS}
      ETHEREUM_HOSTS: ${ETHEREUM_RPC_URL}
      L1_CONSENSUS_HOST_URLS: ${CONSENSUS_BEACON_URL}
      DATA_DIRECTORY: /data
      VALIDATOR_PRIVATE_KEYS: ${VALIDATOR_PRIVATE_KEYS}
      COINBASE: ${COINBASE}
      P2P_IP: ${P2P_IP}
      LOG_LEVEL: info
    entrypoint:
      - sh
      - -c
      - node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network testnet --node --archiver --sequencer
    volumes:
      - /root/.aztec/testnet/data/:/data
```
**👉 Save with `CTRL+X`, press `Y`, then Enter.**

## 7. Run the Node
```bash
docker compose up -d
```
**Check logs:**
```bash
docker compose logs -fn 1000
```
**Done**

## To stop and kill Node
```bash
cd aztec && docker compose down -v
```








