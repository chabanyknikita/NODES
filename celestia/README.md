<div align="center">
  <h1> Manual Installation </h1>
</div>

### Celestia

<figure><img src="../celestia/project_files/ celestia.jpeg" width="150" alt=""><figcaption></figcaption></figure>

**[Website](https://celestia.org/)** | **[Explorer](https://celestia.explorers.guru/)**

### Hardware Requirements

8CPU 16RAM 500GB

**[Rent on Hetzner](https://hetzner.cloud/?ref=AwVksaI2T3Nz)** | **[Rent on Contabo](https://contabo.com/en)**

### Install dependencies

#### Install update, build tools and GO if you needed

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.19.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

### Setup Validator Name

Replace YOUR_MONIKER_NAME with your node name

```
NODE_MONIKER="YOUR_MONIKER_NAME"
```

### Download and build binaries

```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app
git checkout v0.11.0
make install
```

### Initialize the node

```
celestia-appd config chain-id mocha
celestia-appd init "$NODE_MONIKER" --chain-id mocha
```

### Download genesis and addrbook

```
curl -s https://raw.githubusercontent.com/celestiaorg/networks/master/mocha/genesis.json > $HOME/.celestia-app/config/genesis.json
curl -s https://snapshots1-testnet.nodejumper.io/celestia-testnet/addrbook.json > $HOME/.celestia-app/config/addrbook.json

```

### Add seeds

```
SEEDS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mocha/seeds.txt | tr -d '\n')
PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mocha/peers.txt | tr -d '\n')
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.celestia-app/config/config.toml

```

### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.celestia-app/config/config.toml
```

### Set minimum gas price

```
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001utia"|g' $HOME/.celestia-app/config/app.toml
```

### Resetting data

```
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app --keep-addr-book
```

### Create a service

```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null << EOF
[Unit]
Description=Celestia Validator Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which celestia-appd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

### Download snapshot

```
curl -L https://snapshots.kjnodes.com/celestia-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nibid
```

### Start service and check the logs

```
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
sudo journalctl -u celestia-appd -f --no-hostname -o cat
```

### Management

When node is synced you must see **FALSE** after command

```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```

If you see **FALSE**

### Create validator wallet

```
celestia-appd keys add wallet
```

SAVE YOUR INPUT SEED PHRASE (24 words)

### Create ORCHESTRATOR wallet

```
celestia-appd keys add orchestrator
```

SAVE YOUR INPUT SEED PHRASE (24 words)\n
Create ETH address in metamask

### SAVE PRIVATE VALIDATOR KEY

/.andromedad/config/priv_validator_key.json

```
cat /.andromedad/config/priv_validator_key.json
```

### Go to [discord channel](https://discord.gg/kUSueaB22b) #mocha-faucet and paste

```
$request YOUR_VALIDATOR_WALLET_ADDRESS
```

### Create Validator

```
andromedad tx staking create-validator \
--amount=1000000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker="$NODE_MONIKER" \
--chain-id=mocha \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.05 \
--min-self-delegation=1 \
--from=wallet \
--gas-adjustment 1.5 \
--gas auto \
--gas-prices 0.0001utia \
-y
```

### View validator details

```
celestia-appd q staking validator $(celestia-appd keys show wallet --bech val -a)
```
