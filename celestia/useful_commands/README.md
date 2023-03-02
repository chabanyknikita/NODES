## KEY

#### Add new key

```
celestia-appd keys add wallet
```

#### Recover Existing Key

```
celestia-appd keys add wallet --recover
```

#### Query Wallet Balance

```
celestia-appd q bank balances $(celestia-appd keys show wallet -a)
```

## Validator

#### Create new validator

```
celestia-appd tx staking create-validator \
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

#### Edit existing validator

```
celestia-appd tx staking edit-validator \
--moniker="YOUR_MONIKER_NAME" \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL"
--chain-id=mocha \
--commission-rate=0.1 \
--from=wallet \
--gas-adjustment=1.5 \
--gas=auto \
--gas-prices=0.0001utia \
-y
```

#### Unjail validator

```
celestia-appd tx slashing unjail --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### View validator details

```
celestia-appd q staking validator $(celestia-appd keys show wallet --bech val -a)
```

## Tokens

#### Withdraw commission and rewards from your validator

```
celestia-appd tx distribution withdraw-rewards $(celestia-appd keys show wallet --bech val -a) --commission --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Delegate tokens to yourself

```
celestia-appd tx staking delegate $(celestia-appd keys show wallet --bech val -a) 1000000utia --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Delegate tokens to validator

```
celestia-appd tx staking delegate YOUR_TO_VALOPER_ADDRESS 1000000utia --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Unbond tokens from your validator

```
celestia-appd tx staking unbond $(celestia-appd keys show wallet --bech val -a) 1000000utia --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Send tokens to wallet

```
celestia-appd tx bank send wallet YOUR_TO_WALLET_ADDRESS 1000000utia --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

## Governance

#### List all proposals

```
celestia-appd query gov proposals
```

#### View proposal by id

```
celestia-appd query gov proposal 1
```

#### Vote 'Yes'

```
celestia-appd tx gov vote 1 yes --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Vote 'No'

```
celestia-appd tx gov vote 1 no --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Vote 'Abstain'

```
celestia-appd tx gov vote 1 abstain --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

#### Vote 'No With Veto'

```
celestia-appd tx gov vote 1 no_with_veto --from wallet --chain-id mocha --gas-prices 0.1utia --gas-adjustment 1.5 --gas auto -y
```

## Utility

#### Update ports

```
CUSTOM_PORT=26
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}660\"%" $HOME/.celestia-app/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}317\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}091\"%" $HOME/.celestia-app/config/app.toml
```

#### Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.celestia-app/config/app.toml
```

#### Get validator info

```
celestia-appd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
celestia-appd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(celestia-appd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.celestia-app/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Remove node

```
sudo systemctl stop celestia-appd && sudo systemctl disable celestia-appd && sudo rm /etc/systemd/system/celestia-appd.service && sudo systemctl daemon-reload && rm -rf $HOME/.celestia-app && rm -rf $HOME/celestia-app && sudo rm $(which celestia-appd)
```

## Service Management

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable celestia-appd
```

#### Disable service

```
sudo systemctl disable celestia-appd
```

#### Start service

```
sudo systemctl start celestia-appd
```

#### Stop service

```
sudo systemctl stop celestia-appd
```

#### Restart service

```
sudo systemctl restart celestia-appd
```

#### Check service status

```
sudo systemctl status celestia-appd
```

#### Check service logs

```
sudo journalctl -u celestia-appd -f --no-hostname -o cat
```
