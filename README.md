# Relay interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```

## 2. Make home dir for icq and create configurations file
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://127.0.0.1:23657      # use your own GAIA RPC endpoint here
    grpc-addr: http://127.0.0.1:23090     # use your own GAIA GRPC endpoint here
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet
    chain-id: STRIDE-TESTNET-4
    rpc-addr: http://127.0.0.1:16657      # use your own Stride GRPC endpoint here
    grpc-addr: http://127.0.0.1:16090     # use your own Stride GRPC endpoint here
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 3. Import wallets
> NOTE: Please use the same wallet you have used for relayer task, as it is the only way to prove that icq runs on your behalf!
```
icq keys restore --chain stride-testnet wallet
icq keys restore --chain gaia-testnet wallet
```
## 4. Create icq service
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

## 6. Check icq logs
```
journalctl -u icqd -f -o cat
```

You will have to wait some time until you see some logs (5-15min):
```
store/bank/key
height parsed from GetHeightFromMetadata= 0
Fetching client update for height height 176886
store/bank/key
height parsed from GetHeightFromMetadata= 0
Fetching client update for height height 176886
Requerying lightblock
Requerying lightblock
Requerying lightblock
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 176885
Requerying lightblock
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 176885
Send batch of 4 messages
1 ClientUpdate message
1 SubmitResponse message
1 ClientUpdate message
1 SubmitResponse message
Sent batch of 2 (deduplicated) messages
```

After that you can check you transaction in the explorer

![image](https://user-images.githubusercontent.com/50621007/183242421-ca5e8f83-4d54-4ddb-bdbc-31430da23046.png)

# Usefull commands

## Completely remove icq
> NOTE: This will delete your icq from the machine!
```
sudo systemctl stop icqd
sudo systemctl disable icqd
sudo rm /etc/systemd/system/icqd* -rf
sudo rm $(which icq) -rf
sudo rm -rf $HOME/.icq
sudo rm -rf $HOME/interchain-queries
```
