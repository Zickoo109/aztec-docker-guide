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

## 2. Arranging all you need  

- **RPC URLs**: Sepolia RPC + Beacon RPC (from third-party provider or self-hosted)  
- **Wallet private key and public address**: Use a freshly generated EVM wallet for security  
- **Server IP address**  

### Find your server IP
```bash
curl ipv4.icanhazip.com
```

Save it somewhere safe.  

---

## 3. Enable Firewall and Open Ports  

```bash
# Install UFW
sudo apt install -y ufw

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



