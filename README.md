# VPN-сервер VLESS+Reality на Ubuntu

Этот проект демонстрирует настройку и оптимизацию VPN-сервера на базе **Xray** с использованием протокола **VLESS+Reality**.  
Реализация ориентирована на безопасность, обход блокировок и высокую производительность.

---

## 📌 Возможности
- 🔒 Безопасное подключение через **VLESS+Reality**
- 👥 Поддержка нескольких пользователей (UUID)
- 🛡 Усиленная защита сервера:
  - вход только по **SSH-ключам**
  - **UFW firewall**
- ⚡ Оптимизация скорости:
  - **XTLS-Vision + splice**
  - **TCP BBR congestion control**
- 🌀 Fallback-механизм с **Nginx** для маскировки под HTTPS-сайт
- 📊 Логирование и идентификация пользователей через поле `email`

---

## 🚀 Установка

### 1. Обновление системы
```bash
sudo apt update && sudo apt upgrade -y

2. Установка UFW и настройка правил

sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw enable

3. Создание пользователя и защита SSH

sudo adduser deploy
sudo usermod -aG sudo deploy

В /etc/ssh/sshd_config:

PermitRootLogin no
PasswordAuthentication no

Перезапуск:

sudo systemctl restart ssh

4. Установка Xray

bash <(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh) install

5.1 Генерация ключей для Reality

/usr/local/bin/xray x25519

5.2 Генерация UUID

./xray uuid

5.3 Генерация shortId

openssl rand -hex 8

6. Конфигурация Xray

Файл: /usr/local/etc/xray/config.json

{
  "log": { "loglevel": "warning" },
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "uuid-for-user-1",
            "flow": "xtls-rprx-vision",
            "email": "user1@example.com"
          },
          {
            "id": "uuid-for-user-2",
            "flow": "xtls-rprx-vision",
            "email": "user2@example.com"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "www.cloudflare.com:443",
          "serverNames": ["www.cloudflare.com", "dl.google.com"],
          "privateKey": "<your-privateKey>",
          "shortIds": ["your_id"],
          "xver": 0
        }
      },
      "fallbacks": [
        {
          "dest": 80,
          "alpn": "http/1.1",
          "xver": 1
        }
      ]
    }
  ],
  "outbounds": [{ "protocol": "freedom" }]
}

Проверка конфига:

/usr/local/bin/xray run -test -config /usr/local/etc/xray/config.json


---

⚡ Оптимизация TCP (BBR)

sudo modprobe tcp_bbr
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p


---

🌀 Fallback через Nginx

sudo apt install nginx -y

В конфиге Xray fallbacks перенаправляют «левый» HTTPS-трафик на Nginx (порт 80).
Это делает сервер неотличимым от обычного веб-хостинга.


---

📱 Настройка клиента (Nekobox / NekoRay)

Пример клиента (Nekobox JSON):

{
  "server": "your-server-ip",
  "server_port": 443,
  "type": "vless",
  "uuid": "uuid-for-user-1",
  "flow": "xtls-rprx-vision",
  "tls": {
    "enabled": true,
    "reality": {
      "enabled": true,
      "public_key": "<your-publicKey>",
      "short_id": "your_id"
    },
    "server_name": "www.cloudflare.com",
    "utls": {
      "enabled": true,
      "fingerprint": "chrome"
    }
  }
}


---

🔧 Администрирование

Проверка статуса:

systemctl status xray -l

Перезапуск:

sudo systemctl restart xray



---

📊 Логи и мониторинг

Логи ошибок: /var/log/xray/error.log

Для каждого клиента используется поле email, чтобы видеть активность.



---

📌 Автоматическое обновление системы

sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades


---

🏆 Итог

В результате получаем:

Быстрый и скрытный VPN-сервер на базе VLESS+Reality

Защищённый доступ (SSH-ключи, firewall)

Поддержка нескольких пользователей

Маскировка под обычный HTTPS-сайт



---

---
