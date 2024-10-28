# Settlus Node Installation Guide

This guide provides instructions for installing and running a Settlus node.

## System Requirements

- Operating System: Ubuntu 22.04 LTS (recommended)
- CPU: 4 cores or more
- RAM: 8 GB or more 
- Storage: 100 GB SSD or more

## Installation

### 1. Install Dependencies

Update your system and install required packages:

```bash
sudo apt update
sudo apt install build-essential git curl
```

### 2. Install Go

Settlus nodes require Go. Install the latest version:

```bash
wget https://go.dev/dl/go1.19.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.linux-amd64.tar.gz
```

Add Go to your PATH:

```bash
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
source ~/.profile
```

### 3. Clone Settlus Repository

```bash
git clone https://github.com/settlus/chain
cd chain
```

### 4. Build and Install

Compile and install the Settlus node software:

```bash
make install
```

### 5. Initialize the Node

```bash
settlus init <your_node_name> --chain-id settlus_5371-1
```

### 6. Configure the Node

Download the genesis file:

```bash
wget -O $HOME/.settlus/config/genesis.json https://raw.githubusercontent.com/settlus/chain/main/genesis.json
```

Edit the `config.toml` file to add persistent peers:

```bash
nano $HOME/.settlus/config/config.toml
```

### 7. Start the Node

```bash
settlus start
```

## Additional Configuration

- Set up a systemd service for automatic startup
- Configure firewall rules to allow inbound connections on required ports
- Set up monitoring and alerting for your node

## Keeping Your Node Updated

Remember to keep your node updated with the latest releases from the Settlus team.

## Advanced Setup

For more detailed configuration options and advanced setup, refer to the official Settlus documentation.

## Support

If you encounter any issues or have questions, please open an issue in this repository or contact the Settlus support team.

