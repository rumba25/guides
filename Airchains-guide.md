Here's a concise guide to installing an Airchains node:

## Prerequisites

- Minimum hardware requirements: 4 CPU cores, 8GB RAM, 200GB SSD[1]
- Ubuntu 20.04 or later recommended

## Installation Steps

### 1. Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### 2. Install Go

```bash
ver="1.21.6"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### 3. Download and Install Junction Binary

```bash
wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x junctiond
sudo mv junctiond /usr/local/bin/
```

### 4. Initialize the Node

```bash
junctiond init <your_moniker> --chain-id=junction
```

### 5. Configure Genesis and Peers

```bash
wget -O $HOME/.junction/config/genesis.json "https://raw.githubusercontent.com/111STAVR111/props/main/Airchains/genesis.json"
sed -i 's/persistent_peers = ""/persistent_peers = "de2e7251667dee5de5eed98e54a58749fadd23d8@34.22.237.85:26656"/' $HOME/.junction/config/config.toml
```

### 6. Create Service File

```bash
sudo tee /etc/systemd/system/junctiond.service > /dev/null <<EOF
[Unit]
Description=junction
After=network-online.target

[Service]
User=$USER
ExecStart=$(which junctiond) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### 7. Start the Node

```bash
sudo systemctl daemon-reload
sudo systemctl enable junctiond
sudo systemctl start junctiond
```

### 8. Check Node Status

```bash
junctiond status
```

## Becoming a Validator

To become a validator, ensure you have at least 58 tokens in your account[3]. Then create a validator using:

```bash
junctiond tx staking create-validator \
  --amount=1000000amf \
  --pubkey=$(junctiond tendermint show-validator) \
  --moniker="<your_moniker>" \
  --chain-id=junction \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --gas-prices="0.00025amf" \
  --from=<your_wallet_name>
```

