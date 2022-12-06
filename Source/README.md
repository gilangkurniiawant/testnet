<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/189353069-b9796464-574d-4903-b639-163fd0191ec9.png">
</p>

# Source node setup for Testnet — sourcechain-testnet

Thanks to:
>- [NodeX](https://github.com/nodexcapital)

Explorer:
>-  https://explorer.testnet.sourceprotocol.io
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

## Set up your source fullnode
### Automatic Script
You can setup your source fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!
```
wget -O source.sh https://raw.githubusercontent.com/mggnet/testnet/main/Source/source.sh && chmod +x source.sh && ./source.sh
```
## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
sourced status 2>&1 | jq .SyncInfo
```

## SnapShot 27.09.22 (0.1 GB) block height --> 2530401 from Obajay
Source: https://github.com/obajay/StateSync-snapshots/blob/main/Source/README.md
```
sudo systemctl stop sourced
rm -rf $HOME/.source/data/
mkdir $HOME/.source/data/
cd $HOME
wget http://116.202.236.115:7150/sourcedata.tar.gz
tar -C $HOME/ -zxvf sourcedata.tar.gz --strip-components 1
rm sourcedata.tar.gz
sudo systemctl restart sourced && journalctl -u sourced -f -o cat
```

## Quick update (Impotant!)
If the validator was created earlier. Need to reset priv_validator_state.json!, If you dont create validator before please skip this step!
```
wget -O $HOME/.source/data/priv_validator_state.json "https://raw.githubusercontent.com/obajay/StateSync-snapshots/main/priv_validator_state.json"
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
sourced keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
sourced keys add $WALLET --recover
```

To get current list of wallets
```
sourced keys list
```

### Save wallet info
Add wallet and valoper address and load variables into the system
```
SOURCE_WALLET_ADDRESS=$(sourced keys show $WALLET -a)
SOURCE_VALOPER_ADDRESS=$(sourced keys show $WALLET --bech val -a)
echo 'export SOURCE_WALLET_ADDRESS='${SOURCE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SOURCE_VALOPER_ADDRESS='${SOURCE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet
In order to create validator first you need to fund your wallet with testnet tokens.
To top up your wallet join [Source discord server](https://discord.gg/UqzjashS) and navigate to:
- **#faucet** channel

To request a faucet grant:
```
$request <YOUR_WALLET_ADDRESS>
```

### Create validator
Before creating validator please make sure that you have at least 1 qck (1 qck is equal to 1000000 usource) and your node is synchronized

To check your wallet balance:
```
sourced query bank balances $SOURCE_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
sourced tx staking create-validator \
  --amount 1000000usource \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(sourced tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $SOURCE_CHAIN_ID
```

## Security
To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${SOURCE_PORT}656,${SOURCE_PORT}660/tcp
sudo ufw enable
```
### Get list of validators
```
sourced q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${SOURCE_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu sourced -o cat
```

Start service
```
sudo systemctl start sourced
```

Stop service
```
sudo systemctl stop sourced
```

Restart service
```
sudo systemctl restart sourced
```

### Node info
Synchronization info
```
sourced status 2>&1 | jq .SyncInfo
```

Validator info
```
sourced status 2>&1 | jq .ValidatorInfo
```

Node info
```
sourced status 2>&1 | jq .NodeInfo
```

Show node id
```
sourced tendermint show-node-id
```

### Wallet operations
List of wallets
```
sourced keys list
```

Recover wallet
```
sourced keys add $WALLET --recover
```

Delete wallet
```
sourced keys delete $WALLET
```

Get wallet balance
```
sourced query bank balances $SOURCE_WALLET_ADDRESS
```

Transfer funds
```
sourced tx bank send $SOURCE_WALLET_ADDRESS <TO_SOURCE_WALLET_ADDRESS> 10000000usource
```

### Voting
```
sourced tx gov vote 1 yes --from $WALLET --chain-id=$SOURCE_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
sourced tx staking delegate $SOURCE_VALOPER_ADDRESS 10000000usource --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
sourced tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000usource --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
sourced tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
sourced tx distribution withdraw-rewards $SOURCE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$SOURCE_CHAIN_ID
```

### Validator management
Edit validator
```
sourced tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$SOURCE_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
sourced tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$SOURCE_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop sourced
sudo systemctl disable sourced
sudo rm /etc/systemd/system/source* -rf
sudo rm $(which sourced) -rf
sudo rm $HOME/.source* -rf
sudo rm $HOME/source -rf
```
