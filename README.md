# Airchains
**Recommended Hardware Requirements	**

"CPU	4 Cores RAM	8 GB Storage	500 GB NETWORK	1 Gbps OS	Ubuntu 22.04"			

## BUILD BINARY
```
  cd $HOME wget -O junctiond https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond chmod +x junctiond mv junctiond $HOME/go/bin/
```					

## GO
 				
```
cd $HOME && \ ver="1.22.0" && \ wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \ sudo rm -rf /usr/local/go && \ sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \ rm "go$ver.linux-amd64.tar.gz" && \ echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \ source ~/.bash_profile && \ go version
````					

## SET UP SYSTEM
   
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
sudo tee /etc/systemd/system/junction.service > /dev/null << EOF
[Unit]
Description=junction node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.junction"
Environment="DAEMON_NAME=junctiond"
Environment="UNSAFE_SKIP_BACKUP=true"
 
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable junction
```		

## Initialize Node				

  Replace Name with your own moniker  
  
  ```
  MONIKER="Name-your-node"
  ```
  ```
  junctiond init $MONIKER --chain-id junction
  ```				

## LIVE PEER

```
PEERS="47f61921b54a652ca5241e2a7fc4ed8663091e89@airchains-testnet-peer.itrocket.net:19656,e78a440c57576f3743e6aa9db00438462980927e@5.161.199.115:26656,747b20b00224128bb3a3022cfa557fa105a8e41d@84.247.140.127:43456,f786dcc80601ddd33ba98c609795083ba418d740@158.220.119.11:43456,0b1159b05e940a611b275fe0006070439e5b6e69@[2a03:cfc0:8000:13::b910:277f]:13756,c8f6b1a795a6d9cd2ec39faf277163a9711fc81b@38.242.194.19:43456,552d2a5c3d9889444f123d740a20237c89711109@109.199.96.143:43456,cc27f4e54a78b950adaf46e5413f92f5d53d2212@209.126.86.186:43456,f5b69a02abeb3340ccd266f049ed6aabc7c0ea88@94.72.114.150:43456,43ab9af9fabe523a0a8794ae779100c50d1383e6@5.189.142.177:17656,db38d672f66df4de01b26e1fa97e1632fbfb1bdf@173.249.57.190:26656" sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.junction/config/config.toml
````			

## Addrbook	

```
wget -O $HOME/.junction/config/addrbook.json https://testnet-files.itrocket.net/airchains/addrbook.json
```			

## Start Node

```
sudo systemctl start junction journalctl -u junction -f
````	
### Add new wallet					
```
junctiond keys add $WALLET
```					
### Check balance					
```
junctiond q bank balances $WALLET_ADDRESS
```					
### Creat hex to EVM					
```
junctiond keys unsafe-export-eth-key $WALLET
```					
### Creat Validators					
```
junctiond tx staking create-validator \
--amount 1000000amf \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(junctiond tendermint show-validator) \
--moniker "NAME YOUR NODE" \
--identity "ID KEYBASE" \
--details "I love blockchain ❤️" \
--chain-id junction \
--fees 200amf \
-y
````					
### Delegate yourself					
```
junctiond tx staking delegate $(junctiond keys show $WALLET --bech val -a) 1000000amf --from $WALLET --chain-id junction --fees 200amf -y
```
### UNJAIL VALIDATOR
```
junctiond tx slashing unjail --from $WALLET --chain-id junction --fees 200amf -y
```
