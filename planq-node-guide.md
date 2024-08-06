### **Setup validator name**

```bash
MONIKER="Enter Your Moniker Name"
```

### Install dependencies

```bash
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

### Install Go

```bash
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl <https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz> | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

### Install Node

```bash
cd $HOME

rm -rf planq

git clone <https://github.com/planq-network/planq.git>

cd planq

git checkout v1.0.7

make build
```

### Initialize Node

```bash
planqd init "$MONIKER" --chain-id=planq_7070-2
```

### Download Genesis

```bash
curl -Ls <https://snapshots.raylord.best/snapshots/planq/genesis.json> > $HOME/.planqd/config/genesis.json
```

### Download Addrbook

```
curl -Ls <https://snapshots.raylord.best/snapshots/planq/addrbook.json> > $HOME/.planqd/config/addrbook.json
```


### Create Service

```bash
sudo tee /etc/systemd/system/planqd.service > /dev/null <<EOF
[Unit]
Description=planqd Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which planqd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable planqd
```

### Download Snapshot

```bash
planqd tendermint unsafe-reset-all --home $HOME/.planqd --keep-addr-book
curl <https://snapshots.raylord.best/snapshots/planq/snapshot-planq.icebertech.lz4> | lz4 -dc - | tar -xf - -C $HOME/.planqdplanqd
```

### Start the node

```bash
sudo systemctl restart planqd
journalctl -u planqd -f
```


## Snapshot

```bash
sudo systemctl stop planqd

cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup 

planqd tendermint unsafe-reset-all --home $HOME/.planqd --keep-addr-book 
curl <https://snapshots.icebertech.net/snapshots/planq/snapshot-planq.icebertech.lz4> | lz4 -dc - | tar -xf - -C $HOME/.planqd

mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json 

sudo systemctl restart planqd
sudo journalctl -u planqd -f --no-hostname -o cat
```

## State-sync

```bash
sudo systemctl stop planqd

cp $HOME/.planqd/data/priv_validator_state.json $HOME/.planqd/priv_validator_state.json.backup
planqd tendermint unsafe-reset-all --home $HOME/.planqd

peers="4f85f31f8aa696035607dc4596fe426264cc7f02@167.235.14.83:44656"  
SNAP_RPC="<https://rpc.planq.icebertech.net:443>"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \\"$peers\\"/" $HOME/.planqd/config/config.toml 

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000));
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) 

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH && sleep 2

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\\1true| ;
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\\1\\"$SNAP_RPC,$SNAP_RPC\\"| ;
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\\1$BLOCK_HEIGHT| ;
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\\1\\"$TRUST_HASH\\"| ;
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\\1\\"\\"|" $HOME/.planqd/config/config.toml

mv $HOME/.planqd/priv_validator_state.json.backup $HOME/.planqd/data/priv_validator_state.json

sudo systemctl restart planqd && sudo journalctl -u planqd -f
```
