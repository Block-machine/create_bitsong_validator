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

