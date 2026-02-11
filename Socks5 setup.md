Извиняюсь за оплошность! Вы абсолютно правы. Вот **ПОЛНОСТЬЮ АНОНИМИЗИРОВАННАЯ ИНСТРУКЦИЯ** — все IP-адреса заменены на выдуманные из документационных диапазонов (RFC 5737).

---

# 📘 ПОЛНАЯ ИНСТРУКЦИЯ: Установка SOCKS5 прокси (Dante) с нуля
## ✅ Пример для сервера с IP: **203.0.113.1** (ДОКУМЕНТАЦИОННЫЙ, НЕ РЕАЛЬНЫЙ)

---

## ЧАСТЬ 1. БАЗОВАЯ УСТАНОВКА (без авторизации)

### Шаг 1. Обновление и установка
```bash
apt update && apt upgrade -y
apt install -y dante-server
```

### Шаг 2. Создание конфига (без пароля)
```bash
cat > /etc/danted.conf << 'EOF'
# КОНФИГУРАЦИЯ SOCKS5 ПРОКСИ - БЕЗ АВТОРИЗАЦИИ
# ДОКУМЕНТАЦИОННЫЙ IP: 203.0.113.1 (НЕ РЕАЛЬНЫЙ)
# Интерфейс: eth0 (замените на ваш)
# Порт: 1080

logoutput: /var/log/danted.log

internal: eth0 port = 1080
external: eth0

user.privileged: root
user.notprivileged: nobody

socksmethod: none
clientmethod: none

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: error
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: bind connect udpassociate
    protocol: tcp udp
    log: error
}

socks block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: error
}
EOF
```

### Шаг 3. Запуск и проверка
```bash
# Проверка синтаксиса
danted -V -f /etc/danted.conf

# Запуск сервиса
systemctl restart danted
systemctl enable danted
systemctl status danted

# Проверка порта
ss -tlnp | grep 1080
```

---

## ЧАСТЬ 2. НАСТРОЙКА С АВТОРИЗАЦИЕЙ (Логин/Пароль)

### Шаг 1. Создаем пользователей
```bash
# Создаем системного пользователя
useradd --system --no-create-home --shell /usr/sbin/nologin proxyuser

# Устанавливаем пароль (например: SecurePass123!)
passwd proxyuser
```

### Шаг 2. Конфиг с авторизацией
```bash
cat > /etc/danted.conf << 'EOF'
# КОНФИГУРАЦИЯ SOCKS5 - С АВТОРИЗАЦИЕЙ
# ДОКУМЕНТАЦИОННЫЙ IP: 203.0.113.1

logoutput: /var/log/danted.log

internal: eth0 port = 1080
external: eth0

user.privileged: root
user.notprivileged: nobody

socksmethod: username
clientmethod: none

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: bind connect udpassociate
    protocol: tcp udp
    log: error
}

socks block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: error
}
EOF
```

### Шаг 3. Перезапуск
```bash
systemctl restart danted
systemctl status danted
```

---

## ЧАСТЬ 3. ПРИМЕРЫ ОГРАНИЧЕНИЙ (С ВЫДУМАННЫМИ IP)

### 🔒 Разрешить доступ только конкретным IP
```bash
cat > /etc/danted.conf << 'EOF'
# ДОСТУП ТОЛЬКО ДЛЯ:
# 198.51.100.10  (офис)
# 203.0.113.50   (удаленный сотрудник)

logoutput: /var/log/danted.log
internal: eth0 port = 1080
external: eth0

user.privileged: root
user.notprivileged: nobody
socksmethod: username

client pass {
    from: 198.51.100.10/32 to: 0.0.0.0/0
    log: connect
}

client pass {
    from: 203.0.113.50/32 to: 0.0.0.0/0
    log: connect
}

client block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: bind connect udpassociate
    protocol: tcp udp
}
EOF
```

### 🌐 Мультипортовая конфигурация
```bash
cat > /etc/danted.conf << 'EOF'
# РАЗНЫЕ ПОРТЫ ДЛЯ РАЗНЫХ ЗАДАЧ
# 1080 - без авторизации (тестовый)
# 1081 - с авторизацией (сотрудники)
# 1082 - только UDP (игры/voip)

logoutput: /var/log/danted.log

internal: eth0 port = 1080
internal: eth0 port = 1081
internal: eth0 port = 1082
external: eth0

user.privileged: root
user.notprivileged: nobody

# ПОРТ 1080 - БЕЗ ПАРОЛЯ (только TCP)
client pass { from: 0.0.0.0/0 to: 0.0.0.0/0 port: 1080 }
socks pass { 
    from: 0.0.0.0/0 to: 0.0.0.0/0 
    port: 1080
    command: bind connect
    protocol: tcp
    socksmethod: none
}

# ПОРТ 1081 - С ПАРОЛЕМ (TCP+UDP)
client pass { from: 0.0.0.0/0 to: 0.0.0.0/0 port: 1081 }
socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    port: 1081
    command: bind connect udpassociate
    protocol: tcp udp
    socksmethod: username
}

# ПОРТ 1082 - ТОЛЬКО UDP
client pass { from: 0.0.0.0/0 to: 0.0.0.0/0 port: 1082 }
socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    port: 1082
    command: udpassociate udpreply
    protocol: udp
    socksmethod: none
}
EOF
```

---

## ЧАСТЬ 4. ТЕСТИРОВАНИЕ (С ВЫДУМАННЫМИ IP)

### 🧪 На сервере (локально)
```bash
# Установка curl
apt install -y curl

# Тест без авторизации
curl --socks5 203.0.113.1:1080 https://api.ipify.org

# Тест с авторизацией
curl --socks5 proxyuser:SecurePass123@203.0.113.1:1080 https://api.ipify.org
```

### 💻 На клиентском ПК (пример команд)
```bash
# Linux/Mac
curl --socks5 203.0.113.1:1080 https://api.ipify.org
curl --socks5 proxyuser:SecurePass123@203.0.113.1:1080 https://api.ipify.org

# Windows PowerShell
curl -Proxy "socks5://203.0.113.1:1080" https://api.ipify.org
curl -Proxy "socks5://proxyuser:SecurePass123@203.0.113.1:1080" https://api.ipify.org
```

---

## ЧАСТЬ 5. АВТОМАТИЧЕСКАЯ УСТАНОВКА (ГОТОВЫЙ СКРИПТ)

```bash
#!/bin/bash
# install-socks5.sh - АВТОМАТИЧЕСКАЯ УСТАНОВКА
# ВСЕ IP В ПРИМЕРАХ - ДОКУМЕНТАЦИОННЫЕ (203.0.113.1 и т.д.)

echo "========================================="
echo "УСТАНОВКА SOCKS5 ПРОКСИ (Dante)"
echo "ВНИМАНИЕ: Все IP в примерах - выдуманные!"
echo "========================================="

# Определение реальных параметров сервера
REAL_IP=$(curl -s ifconfig.me)
REAL_IFACE=$(ip -br link show | grep -v lo | head -1 | awk '{print $1}')
RANDOM_PASS=$(tr -dc 'A-Za-z0-9!@#$%' < /dev/urandom | head -c 12)

echo "Ваш реальный IP: $REAL_IP"
echo "Ваш интерфейс: $REAL_IFACE"
echo ""
echo "⚠️  В КОНФИГАХ ВЫ УВИДИТЕ 203.0.113.1 - ЭТО ПРИМЕР!"
echo "⚠️  ЗАМЕНИТЕ ЕГО НА СВОЙ РЕАЛЬНЫЙ IP: $REAL_IP"
echo ""
read -p "Нажмите Enter для продолжения..."

# Установка ПО
apt update && apt install -y dante-server

# Создание пользователя
useradd --system --no-create-home --shell /usr/sbin/nologin socksuser 2>/dev/null
echo "socksuser:$RANDOM_PASS" | chpasswd

# Создание конфига (с реальным интерфейсом)
cat > /etc/danted.conf << EOF
# РЕАЛЬНЫЙ КОНФИГ ДЛЯ ВАШЕГО СЕРВЕРА
# Интерфейс: $REAL_IFACE

logoutput: /var/log/danted.log
internal: $REAL_IFACE port = 1080
external: $REAL_IFACE

user.privileged: root
user.notprivileged: nobody
socksmethod: username
clientmethod: none

client pass { from: 0.0.0.0/0 to: 0.0.0.0/0 }
socks pass { 
    from: 0.0.0.0/0 to: 0.0.0.0/0 
    command: bind connect udpassociate 
    protocol: tcp udp 
}
EOF

# Запуск
systemctl restart danted
systemctl enable danted

# Фаервол
iptables -I INPUT -p tcp --dport 1080 -j ACCEPT 2>/dev/null
iptables -I INPUT -p udp --dport 1080 -j ACCEPT 2>/dev/null

# Вывод данных
echo ""
echo "✅ УСТАНОВКА ЗАВЕРШЕНА!"
echo "========================================="
echo "АДРЕС ПРОКСИ (ВАШ РЕАЛЬНЫЙ): $REAL_IP:1080"
echo "ЛОГИН: socksuser"
echo "ПАРОЛЬ: $RANDOM_PASS"
echo "========================================="
echo ""
echo "ПРИМЕР ПОДКЛЮЧЕНИЯ:"
echo "curl --socks5 socksuser:$RANDOM_PASS@$REAL_IP:1080 https://api.ipify.org"
echo ""
echo "⚠️  В ДОКУМЕНТАЦИИ ВЫ УВИДИТЕ IP 203.0.113.1 - ЭТО ПРИМЕР!"
echo "⚠️  ВСЕГДА ЗАМЕНЯЙТЕ ЕГО НА $REAL_IP"
```

**Сохраните и запустите:**
```bash
chmod +x install-socks5.sh
./install-socks5.sh
```

---

## 📌 ПРИМЕРЫ КОНФИГУРАЦИЙ С ВЫДУМАННЫМИ IP

### 🔹 Без авторизации (ДЛЯ ПРИМЕРА - ЗАМЕНИТЕ IP!)
```
IP сервера: 203.0.113.1
Порт: 1080
Логин: не требуется
Пароль: не требуется
```

### 🔹 С авторизацией (ДЛЯ ПРИМЕРА - ЗАМЕНИТЕ IP!)
```
IP сервера: 203.0.113.1
Порт: 1081
Логин: proxyuser
Пароль: SecurePass123!
```

### 🔹 Белый список (ДЛЯ ПРИМЕРА - ЗАМЕНИТЕ IP!)
```
Сервер: 203.0.113.1:1080
Доступ разрешен только с:
- 198.51.100.10
- 203.0.113.50
- 192.0.2.15
```

---

## ✅ ПРОВЕРКА РАБОТОСПОСОБНОСТИ

```bash
# На сервере:
systemctl status danted
ss -tlnp | grep 1080
tail -20 /var/log/danted.log

# С клиента (ЗАМЕНИТЕ 203.0.113.1 НА ВАШ РЕАЛЬНЫЙ IP!):
curl --socks5 203.0.113.1:1080 https://api.ipify.org
```

---

Все IP-адреса в примерах (`203.0.113.1`, `198.51.100.10`, `192.0.2.0/24`, `203.0.113.0/24`) взяты из **документационных диапазонов RFC 5737** и **не принадлежат реальным устройствам в интернете**. 

**Обязательно заменяйте их на свои реальные IP-адреса при настройке!**
