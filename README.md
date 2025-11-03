# Hyperliquid-Non-Validator-Guide (редактируется)
# 🌿 Hyperliquid Node — Пошаговое Руководство 🌿

## 👩‍💻 Этот гайд поможет развернуть **Non-Validator ноду Hyperliquid Testnet** на Ubuntu 24.04 LTS.
---
### 📗 Содержание 🌱

1. [🌟 Требования](#требования)
2. [🛠 Подготовка сервера](#подготовка-сервера)
3. [📥 Установка HL-Visor](#установка-hl-visor)
4. [⚙ Создание конфигурации](#создание-конфигурации)
5. [▶ Запуск ноды вручную](#запуск-ноды-вручную)
6. [⚡ Запуск через systemd](#запуск-через-systemd)
7. [📊 Мониторинг и логирование](#мониторинг-и-логирование)
8. [⚠ Часто встречающиеся ошибки](#часто-встречающиеся-ошибки)
9. [💡 Полезные команды](#полезные-команды)

---
### 🌟 Требования
* Ubuntu 24.04 LTS (или Ubuntu 22.04 с обновлением до 24.04) 🍀
* Минимум 4 CPU, 8 GB RAM, 500 GB свободного диска 💾
* Постоянное подключение к интернету с открытыми портами **4001-4002** 🌐
* Пользователь `root` для упрощения установки 🧠
---
### 🛠 Подготовка сервера
1. Обновляем систему:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget gnupg lsb-release zstd unzip python3 python3-pip -y
```
2. Проверяем версию Ubuntu:
```bash
lsb_release -a
```
#### Если нужно обновить версию Ubuntu 22.04 до 24.0422, можно выполнить принудительный переход через смену репозиториев (надёжно)
Перед обновлением в файле /etc/update-manager/release-upgrades установить Prompt=normal
💡 Рекомендация перед апгрейдом сделать резерв:
```bash
sudo tar czf /root/backup-etc-$(date +%F).tar.gz /etc
```
Если это Proxmox VM — просто создай snapshot средствами Proxmox. После обновления можно будет удалить backup.
####⚠️ Выполняй только если это чистая Ubuntu 22.04, не кастомизированная. 
```bash
sudo sed -i 's/jammy/noble/g' /etc/apt/sources.list
sudo apt update
sudo apt full-upgrade -y
```
#### Во время установки нужно будет ответить на несколько вопрос системы:
1. Restart services during package upgrades without asking? (разрешить ли автоматически перезапускать службы (service), если во время обновления затронуты системные библиотеки (libc, libssl, libpam и т. д.), рекомендуется ответить "Y".
2. Crontab (Y/I/N/O/D/Z) [default=N] ? если у вас были назначены задания в crontab, то отвечаем "N".
3. What do you want to do about modified configuration file sshd_config? Если ты не менял sshd_config локально, безопасно выбрать "Install the package maintainer's version".
#### После установки ядра и пакетов:
```bash
sudo reboot
```
```bash
lsb_release -a
```
Ожидаемый вывод:
 - No LSB modules are available.
 - Distributor ID: Ubuntu
 - Description:    Ubuntu 24.04.3 LTS
 - Release:        24.04
 - Codename:       noble
---
### ✅ Установка AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
Ожидаемый вывод:
- aws-cli/2.31.27 Python/3.13.9 Linux/5.15.158-2-pve exe/x86_64.ubuntu.24
После этого смело удаляем скачанный архив:
```bash
rm -f /root/awscliv2.zip
```
---
### 📥 Установка HL-Visor 💚
1. Скачиваем публичный ключ Hyperliquid:
```bash
curl -O https://raw.githubusercontent.com/hyperliquid-dex/node/main/pub_key.asc
gpg --import pub_key.asc
```
2. Скачиваем бинарник HL-Visor и проверяем подпись:
```bash
curl -L https://binaries.hyperliquid-testnet.xyz/Testnet/hl-visor -o hl-visor
curl -L https://binaries.hyperliquid-testnet.xyz/Testnet/hl-visor.asc -o hl-visor.asc
gpg --verify hl-visor.asc hl-visor
chmod a+x hl-visor
```
Если увидишь что-то подобное, то значит подпись корректная и файл не был изменён.
- gpg: Signature made Mon Nov 3 07:20:06 2025 UTC gpg: using EDDSA key CF2C2EA3DC3E8F042A55FB6503254A9349F1820B
- gpg: Good signature from "Hyperliquid <notices@hyperfoundation.org>" [unknown]
- gpg: WARNING: This key is not certified with a trusted signature!
- gpg: There is no indication that the signature belongs to the owner.
- Primary key fingerprint: CF2C 2EA3 DC3E 8F04 2A55 FB65 0325 4A93 49F1 820B"

---
### ⚙ Создание конфигурации
1. Создаём папку и конфиг:
```bash
mkdir -p ~/hl
echo '{"chain": "Testnet"}' > ~/hl/visor.json
```
2. Делаем бинарник исполняемым:
```bash
chmod a+x ~/hl-visor
```
3. Создаём конфигурационный файл
```bash
nano /root/hl/visor.json
```
---
### ⚡ Запуск через systemd 🌱

1. Создаём файл `/etc/systemd/system/hl-visor.service`:
```bash
```ini
[Unit]
Description=Hyperliquid hl-visor Non-Validator
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/hl
ExecStart=/root/hl/hl-visor run-non-validator --write-trades --write-order-statuses --serve-eth-rpc
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
2. Перезагружаем systemd и запускаем сервис:
```bash
systemctl daemon-reload
systemctl enable --now hl-visor.service
systemctl status hl-visor -l
```
#### 📘 Пояснение к флагам:
> --write-trades — сохраняет сделки
> --write-order-statuses — статус ордеров
> --write-fills — заполнения (fill events)
> --serve-evm-rpc — включает RPC-интерфейс (например, для внешних запросов)
> --serve-info — запускает информационный API
> --replica-cmds-style actions — обязательный параметр, без него не запустится
---
#### Если бакет открыт, можно попробовать скачать оттуда напрямую:
```bash
cd /root/hl/hyperliquid_data
curl -s https://hyperliquid-archive.s3.amazonaws.com/Testnet/snapshot/latest.tar.zst -o latest.tar.zst
```
## ▶ Запуск ноды вручную 😘 (по необходимости)
```bash
cd ~
./hl-visor run-non-validator --write-trades --write-order-statuses --serve-eth-rpc
``

## Запуск visor в валидаторском режиме 💚

1️⃣ Генерация ключа валидатора

Если у тебя ещё нет ключа, обычно делают так (в примере через hl-visor CLI):

```bash
./hl-visor generate-keys
```
Это создаёт пару ключей:

Приватный — хранится у тебя и никому не показывай

Публичный — используется в сети, чтобы тебя идентифицировать как валидатора

Обычно файлы сохраняются в:

~/hl/keys/validator_priv.key
~/hl/keys/validator_pub.key

Тогда для .env ты указываешь путь или сам приватный ключ в переменной:

VALIDATOR_PRIVATE_KEY=$(cat ~/hl/keys/validator_priv.key)

2️⃣ Если нода non-validator

Для тестовой сети или просто синхронизации данных приватный ключ не нужен, переменную можно оставить пустой:

VALIDATOR_PRIVATE_KEY=


Тогда нода работает в режиме non-validator, просто подключается к пирами и скачивает блоки.

💡 Совет: ключ валидатора — как банковский пароль для сети. Никому его не показывай, и резервную копию держи в безопасном месте.

Запуск в валидаторском режиме (вручную, чтобы не путать systemd).

```bash
cd /root/hl
./hl-visor run-validator \
  --config /root/hl/non_validator_config.json \
  --visor /root/hl/visor.json \
  --gossip /root/hl/override_gossip_config.json
```
---
* Первая синхронизация может занять **30–60 минут** ⏳
* Ошибки вида `missing file: visor_abci_state.json` — **нормальные на старте** 😘🍃  
##  📊 Мониторинг и логирование 🌱

1. Просмотр логов в реальном времени:

```bash
journalctl -u hl-visor -f
```

2. Проверка подключённых peers:

```bash
journalctl -u hl-visor | grep "connected to abci stream from" | awk '{print $NF}' | sort | uniq | wc -l
```

3. Файл `visor_abci_state.json` создаётся автоматически после синхронизации 🌱

---

## ⚠ Часто встречающиеся ошибки 💚

* `missing file: visor_abci_state.json` — **норма на старте** 🍃
* `early eof` / `no quorum app hash` — нода ещё **не синхронизировалась**
* `Failed to determine user credentials` — убедитесь, что в systemd указано `User=root` 🧠

---

## 💡 Полезные команды 😘🧴

```bash
1. Количество активных peers
journalctl -u hl-visor | grep "connected to abci stream from" | awk '{print $NF}' | sort | uniq | wc -l

2. 
systemctl restart hl-visor
systemctl stop hl-visor
systemctl status hl-visor -l
journalctl -u hl-visor -f
```

---

> 😘 Пусть ваша нода всегда будет синхронизирована, а сеть — стабильной! ✨
