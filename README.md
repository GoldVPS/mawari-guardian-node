# Mawari Guardian Node (Testnet) — Decentralized Infrastructure Offering (DIO)
Ubuntu 24.04 (Noble) — Plain Text Setup Guide

## System Requirements

|  **Recommended**  |
|-------------------|
|       4 vCPU      |
|      8 GB RAM     |
|      50GB SSD     |

## Get a Server (Powered by GoldVPS)

Need a reliable VPS for this node?  
**GoldVPS** offers SSD NVMe servers optimized for Docker.

- ✅ Quick deployment on Ubuntu 24.04
- ✅ Stable bandwidth
- ✅ NVMe SSD

## **You can purchase here:** https://goldvps.net  
Or contact us on Telegram: https://t.me/miftaikyy


## 0) Prepare Wallet, Tokens, and Guardian NFTs (in your browser)
1. Open https://testnet.mawari.net/mint and connect your wallet. Switch to Mawari Network TestNet.
2. Faucet tokens: go to https://hub.testnet.mawari.net/ and request MAWARI test tokens (max 2) for your MAIN wallet address.
3. Back on https://testnet.mawari.net/, mint up to 3 Guardian NFTs to your MAIN wallet.
Note: Later you will delegate these NFTs to the burner/operator wallet created by the node.

## 1) Install Docker (Ubuntu 24.04)
#### Run these commands one by one:

```bash
sudo apt-get update -y
sudo apt-get install -y ca-certificates curl gnupg
```

  ```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

  ```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

  ```bash
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Verify
  ```bash
docker --version
```

## 2) Run the Guardian Node (Docker)
#### Set OWNER_ADDRESS to your MAIN wallet address (the one that owns the Guardian NFTs).
##### the node will generate a burner wallet automatically.

```bash
export MNTESTNET_IMAGE=us-east4-docker.pkg.dev/mawarinetwork-dev/mwr-net-d-car-uses4-public-docker-registry-e62e/mawari-node:latest
export OWNER_ADDRESS=0xYOUR_OWNER_ADDRESS
```

#### Create cache folder and run the container (daemon + auto-restart)
```bash
mkdir -p ~/mawari
docker run -d --name mawari-guardian   --pull always   --restart unless-stopped   -v ~/mawari:/app/cache   -e OWNERS_ALLOWLIST=$OWNER_ADDRESS   $MNTESTNET_IMAGE
```

#### Follow logs (Ctrl+C to exit)
```bash
docker logs -f mawari-guardian
```

#### Expected early log lines include:
- generating burner wallet
- Using burner wallet {"address": "0x..."}  <-- this is the burner/operator ADDRESS

#### You can print just the burner address line:
```bash
docker logs mawari-guardian | grep -i "Using burner wallet"
```
## 3) Fund the Burner Wallet with 1 Test Token
#### - If you requested 2 tokens to your MAIN wallet earlier, send 1 token to the burner wallet address.
#### - If you only have 1 token, request an additional token at https://hub.testnet.mawari.net/ directly for the burner address.

## 4) Delegate Guardian NFTs to the Burner Wallet
#### 1. Open the Guardian Dashboard: https://app.testnet.mawari.net/licenses
#### 2. Connect your MAIN wallet (where the NFTs were minted).
#### 3. Select all Guardian NFT IDs → click “Delegate” → paste the burner wallet address → confirm “Delegate”.
#### 4. Sign the transaction(s) in your wallet.

Check the container logs again. A healthy delegation sequence looks like:
- received delegation offers count {"delegation offers": "1"}
- accepting delegation offer {...}
- transaction submitted {"hash": "0x..."}
- delegation offer accepted {...}

## 5) CRITICAL: Back Up the Burner/Operator PRIVATE KEY

#### Extract and save the burner private key securely
```bash
  cat ~/mawari/flohive-cache.json | grep -i "privateKey"
```
#### Important: Always treat the burner private key as sensitive. The burner receives delegated NFTs and tokens.
Losing control of this key means losing control of your node’s operator wallet.

## 6) Daily Operations (cheat sheet)
#### Show running containers
```bash
docker ps
```

#### Tail logs
```bash
docker logs -f mawari-guardian
```

#### Stop / start / restart
```bash
docker stop mawari-guardian
docker start mawari-guardian
docker restart mawari-guardian
```

#### Update to latest image (cache and burner wallet persist in ~/mawari)
```bash
docker pull $MNTESTNET_IMAGE
docker rm -f mawari-guardian
docker run -d --name mawari-guardian   --pull always --restart unless-stopped   -v ~/mawari:/app/cache   -e OWNERS_ALLOWLIST=$OWNER_ADDRESS   $MNTESTNET_IMAGE
```
Notes and Tips
- OWNER_ADDRESS is only needed at container creation time. If you recreate the container in a new shell session, export it again before docker run.
- If you want to reset and generate a BRAND NEW burner wallet, stop the container and remove/backup the cache JSON (e.g., flohive-cache.json), then start the container again. You must faucet and delegate again to the NEW burner address.
- Keep ~/mawari backed up (or at least the burner private key) before major changes.
- Secure your server (SSH keys, firewall, regular updates). Never expose private keys in logs, screenshots, or public repos.

Troubleshooting (brief)
- Docker permission denied: run as root or add your user to the docker group and re-login:
  sudo usermod -aG docker $USER

- Image pull failures: check connectivity/DNS/firewall to us-east4-docker.pkg.dev, then retry:
  docker pull $MNTESTNET_IMAGE

- No burner address in logs: ensure the ~/mawari directory is writable and mounted to /app/cache, then restart and check logs:
  ls -l ~/mawari
  docker logs mawari-guardian | grep -i burner

- Delegation not detected: verify you delegated FROM MAIN wallet TO the burner address (not the other way around). Ensure the burner holds 1 test token and wait a short while while watching the logs.
