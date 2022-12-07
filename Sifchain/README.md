<p align="center">
  <img height="300" height="auto" src="https://user-images.githubusercontent.com/44331529/190616339-2de8f67c-4818-4a99-8b5b-6b6e713fd023.png">
</p>

# SIFCHAIN node setup for Testnet

Thanks to:
>- [STAVR](https://github.com/obajay)

Explorer:
>-  https://www.mintscan.io/sifchain/validators

## Hardware Requirements

### Minimum Hardware Requirements
 - 3x CPUs; the faster clock speed the better
 - 4GB RAM
 - 80GB Disk
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Set up your Node 👇
### Automatic Script
```bash
wget -O sifchain https://raw.githubusercontent.com/mggnet/testnet/main/Sifchain/sifchain && chmod +x sifchain && ./sifchain
```

# 2) Manual installation

### Preparing the server

    sudo apt update && sudo apt upgrade -y
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## GO 18.5

```bash
cd $HOME
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Build 01.12.22
```python
cd $HOME
wget https://github.com/Sifchain/sifnode/releases/download/v1.1.0-beta/sifnoded-v1.1.0-beta-linux-amd64.zip
unzip sifnoded-v1.1.0-beta-linux-amd64.zip
rm -rf sifnoded-v1.1.0-beta-linux-amd64.zip
mv ~/sifnoded /usr/local/bin/
```
`sifnoded version`
- 1.1.0-beta

```bash
sifnoded init <YOUR_NODE_NAME> --chain-id sifchain-1
```    

## Create/recover wallet
```bash
sifnoded keys add <walletname>
sifnoded keys add <walletname> --recover
```

## Download Genesis

```bash
cd "${HOME}"/.sifnoded/config
wget -O genesis.json.gz https://raw.githubusercontent.com/Sifchain/networks/master/betanet/sifchain-1/genesis.json.gz
gunzip genesis.json.gz
```

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```
sed -i "s/persistent_peers =.*/persistent_peers = \"0d4981bdaf4d5d73bad00af3b1fa9d699e4d3bc0@44.235.108.41:26656,bcc2d07a14a8a0b3aa202e9ac106dec0bef91fda@13.55.247.60:26656,663dec65b754aceef5fcccb864048305208e7eb2@34.248.110.88:26656,0120f0a48e7e81cc98829ef4f5b39480f11ecd5a@52.76.185.17:26656,6535497f0152293d773108774a705b86c2249a9c@44.238.121.65:26656,fdf5cffc2b20a20fab954d3b6785e9c382762d14@34.255.133.248:26656,8c240f71f9e060277ce18dc09d82d3bbb05d1972@13.211.43.177:26656,9fbcb6bd5a7f20a716564157c4f6296d2faf5f64@18.138.208.95:26656\"/g" "${HOME}"/.sifnoded/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0stake\"/;" ~/.sifnoded/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sifnoded/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sifnoded/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sifnoded/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.sifnoded/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.sifnoded/config/config.toml

```
### Pruning (optional)

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.sifnoded/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.sifnoded/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.sifnoded/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.sifnoded/config/app.toml

### Indexer (optional) 

    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sifnoded/config/config.toml
 
## Download addrbook
```bash
wget -O $HOME/.sifnoded/config/addrbook.json "https://raw.githubusercontent.com/mggnet/testnet/main/Sifchain/addrbook.json"
```

# StateSync
```bash
SNAP_RPC="http://sifchain.rpc.m.stavr.tech:13157"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="cc734391d366c30bf6755dddd07838cfa8731665@135.181.5.47:13156"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.sifnoded/config/config.toml
sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.sifnoded/config/config.toml
sifnoded tendermint unsafe-reset-all --home $HOME/.sifnoded --keep-addr-book
sudo systemctl restart sifnoded && journalctl -u sifnoded -f -o cat
```
# SnapShot (~2.6 GB) updated every 15 hours
```python
cd $HOME
sudo systemctl stop sifnoded
cp $HOME/.sifnoded/data/priv_validator_state.json $HOME/.sifnoded/priv_validator_state.json.backup
rm -rf $HOME/.sifnoded/data
wget http://sifchain.snapshot.stavr.tech:5109/sifchain/sifchain-snap.tar.lz4 && lz4 -c -d $HOME/sifchain-snap.tar.lz4 | tar -x -C $HOME/.sifnoded --strip-components 2
rm -rf sifchain-snap.tar.lz4
mv $HOME/.sifnoded/priv_validator_state.json.backup $HOME/.sifnoded/data/priv_validator_state.json
sudo systemctl restart sifnoded && journalctl -u sifnoded -f -o cat
```

# Create a service file
```bash
sudo tee /etc/systemd/system/sifnoded.service > /dev/null <<EOF
[Unit]
Description=sifchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sifnoded) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable sifnoded && \
sudo systemctl restart sifnoded && \
sudo journalctl -u sifnoded -f -o cat
```

### Create validator
    sifnoded tx staking create-validator \
    --amount=1000000rowan \
    --broadcast-mode=block \
    --pubkey=`sifnoded tendermint show-validator` \
    --moniker=<YOUR_NODE_NAME> \
    --commission-rate="0.1" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.1" \
    --min-self-delegation="1" \
    --from=<walletName> \
    --chain-id=sifchain-1 \
    --gas=auto -y


## Delete node
```bash
sudo systemctl stop sifnoded && \
sudo systemctl disable sifnoded && \
rm /etc/systemd/system/sifnoded.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sifnode && \
rm -rf .sifnoded && \
rm -rf $(which sifnoded)
```
