# Hyperliquid-Non-Validator-Guide
# 🌿 Hyperliquid Non-Validator Node — Пошаговое Руководство 🌿

### 👩‍💻 Этот гайд поможет развернуть **Non-Validator ноду Hyperliquid Testnet** на Ubuntu 24.04 LTS.
---
## 📗 Содержание 🌱

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

## 🌟 Требования

* Ubuntu 24.04 LTS (или Ubuntu 22.04 с обновлением до 24.04) 🍀
* Минимум 4 CPU, 8 GB RAM, 50 GB свободного диска 💾
* Постоянное подключение к интернету с открытыми портами **4001-4002** 🌐
* Пользователь `root` для упрощения установки 🧠
---
## 🛠 Подготовка сервера

1. Обновляем систему:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget gnupg lsb-release -y
```

2. Проверяем версию Ubuntu:

```bash
lsb_release -a
```

---

## 📥 Установка HL-Visor 💚

1. Скачиваем публичный ключ Hyperliquid:

```bash
curl -O https://raw.githubusercontent.com/hyperliquid-dex/node/main/pub_key.asc
gpg --import pub_key.asc
```

2. Скачиваем бинарник HL-Visor и проверяем подпись:

```bash
Скачиваем публичный ключ Hyperliquid:

curl -L https://binaries.hyperliquid-testnet.xyz/Testnet/hl-visor -o hl-visor
curl -L https://binaries.hyperliquid-testnet.xyz/Testnet/hl-visor.asc -o hl-visor.asc
gpg --verify hl-visor.asc hl-visor
chmod a+x hl-visor
```

---

## ⚙ Создание конфигурации

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

* Первая синхронизация может занять **30–60 минут** ⏳
* Ошибки вида `missing file: visor_abci_state.json` — **нормальные на старте** 😘🍃

---

## ⚡ Запуск через systemd 🌱

1. Создаём файл `/etc/systemd/system/hl-visor.service`:

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

```bash

## Если бакет открыт, можно попробовать скачать оттуда напрямую:

cd /root/hl/hyperliquid_data
curl -s https://hyperliquid-archive.s3.amazonaws.com/Testnet/snapshot/latest.tar.zst -o latest.tar.zst

```
## ▶ Запуск ноды вручную 😘 (по необходимости)

```bash
cd ~
./hl-visor run-non-validator --write-trades --write-order-statuses --serve-eth-rpc

```
## 📊 Мониторинг и логирование 🌱

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

> 🎀 Гайд создан с заботой 💚🌿
> 😘 Пусть ваша нода всегда будет синхронизирована, а сеть — стабильной! ✨
