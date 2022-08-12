# Гайд по созданию валидтора в основной сети Bitsong

Ссылка на эксплорер https://explorebitsong.com/bitsong
Демон - bitsongd
ID сети: bitsong-2b
Официальная инструкция: https://docs.bitsong.io/blockchain/create-validator

## Подготовка сервера

Обновляем репозитории

`sudo apt update && sudo apt upgrade -y`

Устанавливаем необходимые утилиты

`sudo apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y`

Устанавливаем Go

```
На примере версии go1.18.3

Скачиваем:
wget https://go.dev/dl/go1.18.3.linux-amd64.tar.gz

Разархивируем: 
sudo tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz

Создаем дирректории для Go:
mkdir -p $HOME/go/bin

Добавляем путь для переменной PATH (необходимо для многоразового использования):
PATH=$PATH:/usr/local/go/bin 

Дозаписываем путь Go в файл .bash_profile (необходимо для многоразового использования):
echo "export PATH=$PATH:$(go env GOPATH)/bin" >> ~/.bash_profile 

Запускаем команды из файла
source ~/.bash_profile

Проверить командой:
go version 

На выходе должны получить такое значение:
"go version go1.18.3 linux/amd64"
```
## Установка ноды

Ставим бинарник

```
cd $HOME
git clone https://github.com/bitsongofficial/go-bitsong
cd go-bitsong
git checkout v0.11.0
make install
```

Проверяем версию

`bitsongd version`

Инициализируем ноду

`bitsongd init <your_custom_moniker>`

Меняем минимальную цену за газ в файле `~/.bitsongd/config/app.toml`

`minimum-gas-prices = "0.0025ubtsg"`

Установка и настройка ноды завершена на этом этапе.

## Настраиваем конфигурацию ноды

Скачиваем генезис-файл
`wget -O ~/.bitsongd/config/genesis.json https://raw.githubusercontent.com/bitsongofficial/networks/master/bitsong-2b/genesis.json`

Добавляем пири в файл $HOME/.bitsongd/config/config.toml
`export PEERS="e2b9971222adf71f7199c670b7e85471c447e926@157.90.255.143:26656,120740c15a8a19c232b1aa4d80b20de248b33db3@135.181.129.94:26656,d741773bc5eecbefb7b14fcca5e3e0fedd49d5a3@157.90.95.104:26656,6e93a30587671e2cecacbcbb27092809bb20249f@95.217.203.59:31656,adfe1cf240780cf8d58266171ced72fb4e9a7a6d@23.226.14.168:26656,f36d3a926ae0583e60f00e7bc54711f3cb7fe769@195.201.58.166:26656,9c9f030298bdda9ca69de7db8e9a3aef33972fba@135.181.16.236:31656,9806602afb65ba45d1048d65285d5c6e50285088@178.18.242.242:26656,4fdd438ea70927003022ecc308e36bc1924ec598@51.210.104.207:26656,3cf3effd3ecb33bdbb5c5e6528c88fde4869b97c@116.202.139.113:26656,075cf589e44c74687ef3a4df3a583f482bce57e0@46.166.143.79:26656,f9d318eaf38988ce2b65b795068d86b214866c91@141.94.170.26:26256,fa932748b327fdde6d235b28a9850f8b8bd3326a@95.217.119.101:31656,d52f6e4fe1819133474e977d7e1d73124d1f4af5@95.217.156.76:26656,5ebab02914638005773dac8026f441e06c115a44@74.207.226.176:26656,e5428ce29ccd26434828a577906ac9c413ca6a48@80.71.57.42:26656,2afc435e2246ff3f16ade85b52264367945d12b5@176.58.124.226:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.bitsongd/config/config.toml`

Дальше синхронизируем ноду командами, используя сейт синк https://medium.com/tendermint/tendermint-core-state-sync-for-developers-70a96ba3ee35

```

sudo systemctl stop bitsong && bitsongd unsafe-reset-all

SNAP_RPC="https://rpc.bitsong.forbole.com:443"
SNAP_RPC2="https://bitsong.stakesystems.io:2053"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

peers="e2b9971222adf71f7199c670b7e85471c447e926@157.90.255.143:26656,120740c15a8a19c232b1aa4d80b20de248b33db3@135.181.129.94:26656,bbfb37b3c44c8148b6af7adfa016ec8fabff69d1@121.78.247.243:16656,d741773bc5eecbefb7b14fcca5e3e0fedd49d5a3@157.90.95.104:26656,6e93a30587671e2cecacbcbb27092809bb20249f@95.217.203.59:31656,adfe1cf240780cf8d58266171ced72fb4e9a7a6d@23.226.14.168:26656,f36d3a926ae0583e60f00e7bc54711f3cb7fe769@195.201.58.166:26656,9c9f030298bdda9ca69de7db8e9a3aef33972fba@135.181.16.236:31656,9806602afb65ba45d1048d65285d5c6e50285088@178.18.242.242:26656,4fdd438ea70927003022ecc308e36bc1924ec598@51.210.104.207:26656,3cf3effd3ecb33bdbb5c5e6528c88fde4869b97c@116.202.139.113:26656"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bitsongd/config/config.toml

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC2\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bitsongd/config/config.toml

sudo systemctl restart bitsong

sudo journalctl -u bitsong -f
```

Запуск в фоне

```
sudo tee /etc/systemd/system/bitsongd.service > /dev/null <<EOF  
[Unit]
Description=BitSong Network Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bitsongd) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

Настраиваем демона
```
sudo -S systemctl daemon-reload
sudo -S systemctl enable bitsongd
```

Запускаем процесс и проверям статус
```
sudo -S systemctl start bitsongd
sudo service bitsongd status
```

## Создание валидатора

Создаем или восстанавливаем кошелек
```
Создать кошелек (обязательно сохраняем seed)
bitsongd keys add <name_wallet> 

Восстановить кошелек (после команды вставить seed) 
bitsongd keys add <name_wallet> --recover
```

Команда создания валидатора
```
bitsongd tx staking create-validator \
    --amount=5000000ubtsg \
    --pubkey=$(bitsongd tendermint show-validator) \
    --moniker="<название валидатора>" \
    --chain-id=bitsong-1 \
    --from=<кошелёк> \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="1"
```

Проверяем, активен ли валидатор
```
bitsongd query tendermint-validator-set | grep "$(bitsongd tendermint show-validator)"
```

Всё готово

## Добавляем лого к своему валидатору
* Клонируем к себе в гитхаб репозиторий - https://github.com/cosmostation/cosmostation_token_resource
* Находим нужную сеть в папке moniker
* Через add file/upload file добавляем своё лого, название файла обязательно должно быть valoperadress.png и только png
* Pull request
