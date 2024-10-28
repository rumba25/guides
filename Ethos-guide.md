```markdown
# Ethos Node Installation Guide

This guide provides step-by-step instructions for installing and configuring an Ethos node.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Advanced Configuration](#advanced-configuration)
- [Monitoring and Management](#monitoring-and-management)
- [Troubleshooting](#troubleshooting)
- [Custom Scripts](#custom-scripts)
- [Remote Management](#remote-management)

## Prerequisites

Before beginning the installation process, ensure you have:

- A compatible system meeting the minimum hardware requirements
- Administrative access to the machine
- A stable internet connection

## Installation Steps

### 1. Download EthOS

1. Visit the official EthOS website and download the latest EthOS installation file.
2. Choose the appropriate version for your system architecture (32-bit or 64-bit).

### 2. Prepare Installation Media

1. Write the EthOS ISO to a USB drive or SSD using a tool like HDD Raw Copy Tool.
2. Select the downloaded EthOS ISO file as the source.
3. Choose your target drive (USB or SSD) as the destination.
4. Click "Start" and wait for the process to complete.

### 3. Boot from Installation Media

1. Connect the USB drive or SSD to your target machine.
2. Restart the computer and boot from the installation media.
3. You may need to adjust BIOS settings to prioritize booting from USB/SSD.

### 4. Initial Setup

Once booted into EthOS:

1. Update EthOS to the latest version:
   ```bash
   sudo ethos-update && sleep 5 && sudo reboot
   ```

2. Set EthOS to single rig mode:
   ```bash
   echo -n "" > /home/ethos/remote.conf
   ```

3. Change the default password:
   ```bash
   passwd
   ```

### 5. Configure Mining Settings

1. Edit the local configuration file:
   ```bash
   nano /home/ethos/local.conf
   ```

2. Set your mining pool and wallet address:
   ```
   proxypool1 eu1.ethermine.org:4444
   proxywallet YourEthereumWalletAddress
   ```

3. Save changes (Ctrl+O, then Enter) and exit (Ctrl+X).

## Advanced Configuration

### Custom Miner Configuration

Edit `local.conf` for more detailed settings:

```
globalminer ethminer
maxgputemp 80
stratumproxy enabled
proxywallet 0x1234567890123456789012345678901234567890
proxypool1 eth-us-east1.nanopool.org:9999
proxypool2 eth-us-west1.nanopool.org:9999
flags --cl-global-work 8192 --farm-recheck 200
```

### GPU Overclocking

Add these lines to `local.conf` for GPU overclocking:

```
globalcore 1200
globalmem 2000
globalfan 75
globalpowertune 20
```

## Monitoring and Management

### Check Mining Status

```bash
show stats
```

### View GPU Information

```bash
show gpus
```

### Restart Mining Process

```bash
minestop
minestart
```

## Troubleshooting

### Clear GPU Settings

```bash
clear-thermals
```

### Rebuild OpenCL Cache

```bash
sudo ethos-overclock
```

## Custom Scripts

### Auto-Reboot Script

Create `/home/ethos/autoreboot.sh`:

```bash
#!/bin/bash

while true; do
    if ! pgrep ethminer > /dev/null; then
        echo "Miner not running. Rebooting..."
        sudo reboot
    fi
    sleep 300
done
```

Make it executable and add to crontab:

```bash
chmod +x /home/ethos/autoreboot.sh
(crontab -l 2>/dev/null; echo "@reboot /home/ethos/autoreboot.sh") | crontab -
```

### Temperature Monitoring Script

Create `/home/ethos/temp_monitor.sh`:

```bash
#!/bin/bash

MAX_TEMP=80
SLEEP_TIME=60

while true; do
    temps=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits)
    for temp in $temps; do
        if [ $temp -gt $MAX_TEMP ]; then
            echo "GPU temperature $temp exceeds maximum. Shutting down miner."
            minestop
            exit 1
        fi
    done
    sleep $SLEEP_TIME
done
```

Make it executable and run in background:

```bash
chmod +x /home/ethos/temp_monitor.sh
nohup /home/ethos/temp_monitor.sh &
```

## Remote Management

### SSH Key-Based Authentication

Generate SSH key pair on local machine:

```bash
ssh-keygen -t rsa -b 4096
```

Copy public key to Ethos node:

```bash
ssh-copy-id ethos@your_ethos_ip
```

SSH without password:

```bash
ssh ethos@your_ethos_ip
```

## Contributing

Feel free to contribute to this guide by submitting pull requests or opening issues for suggestions and improvements.


