
نصب پیش نیاز ها

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

```
ver="1.21.3" &&
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" &&
rm "go$ver.linux-amd64.tar.gz" &&
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile &&
source $HOME/.bash_profile &&
go version

mkdir -p $HOME/go/bin
```


نصب airchains binary

```
cd $HOME
wget -O junctiond https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x junctiond
mv junctiond $HOME/go/bin/
```

کانفیگ کردن نود ***** جای crypton در خط اول اسم خودتون رو بذارید

```
junctiond init crypton --chain-id junction && \
junctiond config set client chain-id junction && \
junctiond config set client keyring-backend test
```

دانلود Genesis . Addressbook

```
curl https://config-t.noders.services/airchains/genesis.json -o ~/.junction/config/genesis.json
curl https://config-t.noders.services/airchains/addrbook.json -o ~/.junction/config/addrbook.json
```

تنظیم پورت ها و ...

```
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"7ebc08bbef4bd2b4da8d881474710a2500854c2b@airchain-t-rpc.noders.services:31656\"/" ~/.junction/config/config.toml

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uamf\"|" ~/.junction/config/app.toml

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  ~/.junction/config/app.toml


echo "export junctiond_PORT="26657"" >> $HOME/.bash_profile

# Set custom ports in app.toml
sed -i.bak -e "s%:1317%:${junctiond_PORT}317%g" \
-e "s%:8080%:${junctiond_PORT}080%g" \
-e "s%:9090%:${junctiond_PORT}090%g" \
-e "s%:9091%:${junctiond_PORT}091%g" \
-e "s%:8545%:${junctiond_PORT}545%g" \
-e "s%:8546%:${junctiond_PORT}546%g" \
-e "s%:6065%:${junctiond_PORT}065%g" ~/.junction/config/app.toml

# Set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${junctiond_PORT}658%g" \
-e "s%:26657%:${junctiond_PORT}657%g" \
-e "s%:6060%:${junctiond_PORT}060%g" \
-e "s%:26656%:${junctiond_PORT}656%g" \
-e "s%:26660%:${junctiond_PORT}660%g" ~/.junction/config/config.toml
```

ساخت سرویس Junctiond

```
sudo tee /etc/systemd/system/junctiond.service > /dev/null << EOF
[Unit]
Description=airchains node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which junctiond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable junction
sudo systemctl enable junctiond

```

فعال سازی و دیدن لاگ

```
sudo systemctl start junctiond && sudo journalctl -u junctiond -f --no-hostname -o cat
```

با کنترل + سی میتونید لاگ رو متوقف کنید



تنها در صورتی که نود شما سینک نمیشد و پیش نمیرفت . **** از یکی از کامند های زیر استفاده کنید تا Peers تغییر کنه و ببینید کدومشون لاگ سالم میندازه -----

Peers : 

```
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"7ebc08bbef4bd2b4da8d881474710a2500854c2b@airchain-t-rpc.noders.services:31656\"/" ~/.junction/config/config.toml

sudo systemctl restart junctiond && sudo journalctl -u junctiond -f --no-hostname -o cat
```


```
PEERS="264493e01774cccdb9baabee4af7146acbec67f2@65.21.193.80:63656,38ffaf594a80b88ffaa0ecb3847bf0f77e5c52fe@5.9.87.231:36656,5880ddf4518b061c111ae6bf07b1ef76ef2a42af@158.220.100.154:26656,d1c949abeb7805546eca0b5e60c4889649760b9c@65.108.121.227:13356,36ed02b04e84fb0ba6382d8d1cd2dbe3195a235b@37.27.64.237:10156,541c513ffc19cff0a02ce4aebaed46f5af020d71@148.251.69.40:22656,dc189095edccb7af8d8c88cd5b9a3ccd25be1115@65.21.237.90:63656,47f61921b54a652ca5241e2a7fc4ed8663091e89@rpc.nodejumper.io:13756,8b2a63f074a37bbfebd82cb78a4893936e1dfd61@37.27.132.57:19656,9ba635344d9c64a4b1d82d7e1138d0216afc27c4@rpc.airchains.aknodes.net:34656,27d1c8383350eb11dc3cbba4b222d4e892e0ec03@45.250.254.41:19656,47f61921b54a652ca5241e2a7fc4ed8663091e89@airchains-testnet-rpc.itrocket.net:19656,4f84487af5e8a86baa7e4e428ca7156ae5bc3ab7@148.251.235.130:24656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.junction/config/config.toml

sudo systemctl restart junctiond && sudo journalctl -u junctiond -f --no-hostname -o cat
```

```
PEERS="47f61921b54a652ca5241e2a7fc4ed8663091e89@airchains-testnet-peer.itrocket.net:19656,c40528f5674b281dfe8a799ba2475782f0b889c7@178.18.240.22:43456,0d03e79ef79687421ac6f4b1ddd6add67dd2d6a0@65.109.83.40:28156,8416ae7dd9a15be93a90b4379e673b71704bde2e@130.185.118.55:656,f9fef92013828d8669712224299136b4632b4904@65.109.113.228:60556,d5ded9ed366f251a59c85f84ed1fa825cceb0d97@[2a01:4f8:221:158e::2]:13656,560162c4502aea50d271b66d220fadeb5cd17038@37.27.68.29:22656,c70f853dba012ee6b6f50070657ca28dbd12daf4@65.108.0.88:13056,7962edd332b1a2293ba6c865aa1f001d6b7cc460@75.119.151.235:43456,0006ec33f419df1530f463cbc18aba9ea9f0abf2@31.220.72.81:26656,aaf57c42eb1a53b487443088db025a5d05f78159@[2a01:4f9:3051:19c2::2]:13756"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.junction/config/config.toml
sudo systemctl restart junctiond && sudo journalctl -u junctiond -f --no-hostname -o cat
```

دیدن لاگ نود -----

Logs :

sudo journalctl -u junctiond -f --no-hostname -o cat


پاک کردن نود -----

```
sudo systemctl stop junctiond
sudo systemctl disable junctiond
sudo rm -rf /etc/systemd/system/junctiond.service
sudo rm $(which junctiond)
sudo rm -rf $HOME/.junction
```

ادامه دارد 





اموزش ساخت استیشن و کسب امتیاز

https://paragraph.xyz/@sarox.eth/airchain-rollup?referrer=0xbefEf0FE13B9bD398A88DAB74CCd62099C51333C


