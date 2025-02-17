# Установка ноды Humans_12.2022

Humans - это суверенная платформа блокчейна нового поколения, которая объединяет экосистему заинтересованных сторон вокруг использования AI для создания масштабных проектов.

Токен уже торгуется, но проект решил переехать с Polkadot на Cosmos. Сейчас запущен тестнет без ревардов, но следующий этап будет награждаемым, а потом выход в мейннет.

По требованиям, это типичный космофорк, так что я запустил валидатора на всякий случай. Поскольку токен уже торгуется, то много ревардов от этого проекта не жду, но место на сервере есть, пускай крутится.



### Эксплоер 

https://explorer.humans.zone/humans-testnet/staking

### Discord 

https://discord.gg/humansdotai

### Сайт проекта (пройдите опрос на сайте, указав почту)
 
https://humans.ai/



- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |


# Ручная установка

### Подготовка сервера

```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19

```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 07.12.22
```python
cd $HOME
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1
go build -o humansd cmd/humansd/main.go
mv humansd /root/go/bin/humansd
```
`humansd version --long`
- version:
- commit: 

```python
humansd config chain-id testnet-1
humansd init STAVRguide --chain-id testnet-1
```    

## Создать/восстановить кошелек
```python
humansd keys add <walletname>
          or 
humansd keys add <walletname> --recover
```

## Скачать Genesis
```python
wget https://snapshots.polkachu.com/testnet-genesis/humans/genesis.json -O $HOME/.humans/config/genesis.json

```
`sha256sum $HOME/.humans/config/genesis.json`
+ f5fef1b574a07965c005b3d7ad013b27db197f57146a12c018338d7e58a4b5cd

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
SEEDS=""
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uheart\"/;" ~/.humans/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.humans/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.humans/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.humans/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.humans/config/config.toml

```
### Pruning (optional)
```python
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.humans/config/app.toml
```
### Indexer (optional) 
```python
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

## Скачать addrbook
```python
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Humans/addrbook.json"
```
[SNAPSHOT](https://polkachu.com/testnets/humans/snapshots)
=

# Создать сервисный файл
```python
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Запуск
```python
sudo systemctl daemon-reload && sudo systemctl enable humansd
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```

### Создаем validator
```python
humansd tx staking create-validator \
  --amount 1000000uheart \
  --from wallet \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(humansd tendermint show-validator) \
  --moniker STAVRguide \
  --chain-id testnet-1 \
  --identity="" \
  --details="" \
  --website="" -y
```

## Удаление node
```bash
sudo systemctl stop humansd && \
sudo systemctl disable humansd && \
rm /etc/systemd/system/humansd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf humans && \
rm -rf .humans && \
rm -rf $(which humansd)
```
#
### Информация о синхронизации
```python
humansd status 2>&1 | jq .SyncInfo
```
### Информация по ноде
```python
humansd status 2>&1 | jq .NodeInfo
```
### Логи ноды
```python
sudo journalctl -u humansd -f -o cat
```
### Баланс кошелька
```python
humansd query bank balances humans...address1yjgn7z09ua9vms259j
```
# Заделегировать себе 1 токен
```humansd tx staking delegate АДРЕС_ВАЛИДАТОРА 1000000uheart --from ИМЯ_ИЛИ_АДРЕС_КОШЕЛЬКА --chain-id testnet-1 --gas-prices 0.1uheart --gas-adjustment 1.5 --gas auto -y ```

# Выход из тюрьмы
```humansd tx slashing unjail --from ИМЯ_ИЛИ_АДРЕС_КОШЕЛЬКА --chain-id testnet-1 --gas-prices 0.1uheart --gas-adjustment 1.5 --gas auto -y```
