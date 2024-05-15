![image](https://github.com/Mozgiii9/WardenProtocolSetupTheNode/assets/74683169/373a9e96-8740-4e14-8283-3c9a19041bd9)

## Обзор проекта Warden Protocol:

**Warden Protocol - проект, созданный на основе Cosmos SDK, который обеспечивает различные уровни выполнения для обеспечения совместимости, модульного управления ключами (этакие инструменты для возможности выбора MPS и HSM операторов) и агрегации учетных записей.**

**Требования к серверу:**

**- CPU: 4 ядра;**
**- RAM: 8GB;**
**- Storage: 200GB SSD(NVME);**
**- OS: Ubuntu 20.04**

## Инструкция по установке ноды Warden Protocol:

**1. Установим обновления пакетов:**

```
sudo apt update && sudo apt upgrade -y
```

**2. Установим необходимое ПО:**

```
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**3. Скачаем скрипт для установки ПО ноды Warden Protocol:**

```
source <(curl -s https://itrocket.net/api/testnet/warden/autoinstall/)
```

**Скрипт откроется автоматически. Указываем имя кошелька, имя ноды(moniker), порт оставляем дефолтным(26). Начнется установка ПО.**

**4. Как только пойдут Height'ы, выходим из меню отображения логов при помощи комбинации клавиш CTRL+C**

**5. Проверим синхронизацию ноды:**

```
wardend status 2>&1 | jq
```

**Необходимо, чтобы статус "catching_up" был "false". Если Ваш статус "catching_up" равен "true", то это говорит о том, что Ваша нода все еще синхронизируется. Если Вы не хотите ждать синхронизации, то Вы можете установить Snapshot на следующем промежуточном шаге**

**5.1(Необязательный шаг). Установка Snapshot'а, чтобы не ждать синхронизацию ноды. Проверяйте актуальный Snapshot на данной [странице](https://itrocket.net/services/testnet/warden/):**

```
sudo systemctl stop wardend
```

```
cp $HOME/.warden/data/priv_validator_state.json $HOME/.warden/priv_validator_state.json.backup
```

```
rm -rf $HOME/.warden/data $HOME/.warden/wasmPath
```

```
curl https://testnet-files.itrocket.net/warden/snap_warden.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.warden
```

```
mv $HOME/.warden/priv_validator_state.json.backup $HOME/.warden/data/priv_validator_state.json
```

```
sudo systemctl restart wardend && sudo journalctl -u wardend -f
```

**6. Еще раз проверим синхронизацию:**

```
wardend status 2>&1 | jq
```

**Статус "catching_up" должен измениться на "false". Если это так, то переходим к следующему шагу**

**7. Создадим кошелек:**

```
wardend keys add $WALLET
```

**Сервер попросит задать пароль для кошелька. Введите его два раза, после чего сохраните mnemonic(seed фразу) и данные от кошелька в надежное место.**

```
WALLET_ADDRESS=$(wardend keys show $WALLET -a)
```

```
VALOPER_ADDRESS=$(wardend keys show $WALLET --bech val -a)
```

```
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
```

```
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
```

```
source $HOME/.bash_profile
```

**8. Зайдем в Keplr и добавим кошелек по seed фразе, которую сохраняли ранее. Переходим в [кран](https://spaceward.alfama.wardenprotocol.org/), подключаем кошелек Keplr и запрашиваем токены. После запроса токенов проверим баланс командой:**

```
wardend query bank balances $WALLET_ADDRESS
```

**9. Создадим валидатора:**

```
cd $HOME
```

```
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(wardend comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000uward\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"I love blockchain \",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
```

**В поле "moniker" замените слово "test" на имя Вашего валидатора**

```
wardend tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id buenavista-1 \
	--gas auto --gas-adjustment 1.5 --fees 600uward \
```

**10. Проверим статус валидатора:**

```
wardend query staking validator $VALOPER_ADDRESS
```

**Набор полезных команд для взаимодействия с нодой:**

**- Проверка информации о ноде:**

```
wardend status 2>&1 | jq
```

**- Проверка логов:**

```
sudo journalctl -u wardend -f
```

**Перезагрузка сервиса:**

```
sudo systemctl restart wardend
```

**Просмотр всех кошельков на сервере:**

```
wardend keys list
```

**Проверка баланса кошелька:**

```
wardend q bank balances $WALLET_ADDRESS
```

**Экспорт кошелька:**

```
wardend keys export $WALLET
```

**Делегировать токены самому себе:**

```
wardend tx staking delegate $(wardend keys show $WALLET --bech val -a) 1000000uward --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 600uward -y 
```

**Делегировать токены другому валидатору:**

```
wardend tx staking delegate <TO_VALOPER_ADDRESS> 1000000uward --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 600uward -y 	
```

**Информация о тюрьме:**

```
wardend q slashing signing-info $(wardend tendermint show-validator) 
```

**Освободить валидатора из тюрьмы:**

```
wardend tx slashing unjail --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 600uward -y 
```

**Удалить ноду:**

```
sudo systemctl stop wardend
```

```
sudo systemctl disable wardend
```

```
sudo rm -rf /etc/systemd/system/wardend.service
```

```
sudo rm $(which wardend)
```

```
sudo rm -rf $HOME/.warden
```

```
sed -i "/WARDEN_/d" $HOME/.bash_profile
```

## Обязательно проведите собственный ресерч проектов перед тем как ставить ноду. Сообщество NodeRunner не несет ответственность за Ваши действия и средства. Помните, проводя свой ресёрч, Вы учитесь и развиваетесь.

## Связь со мной: [Telegram(@M0zgiii)](https://t.me/m0zgiii)
## Мои соц. сети: [Twitter](https://twitter.com/m0zgiii)
