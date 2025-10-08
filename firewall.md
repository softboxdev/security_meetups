
---

# **🔧 САМОСТОЯТЕЛЬНАЯ РАБОТА: НАСТРОЙКА МЕЖСЕТЕВОГО ЭКРАНА НА UBUNTU 24.04**

## **🎯 ЦЕЛЬ РАБОТЫ**
Научиться настраивать и управлять межсетевым экраном на Ubuntu 24.04 с использованием `ufw` и `iptables`.

---

## **📋 ПРЕДВАРИТЕЛЬНЫЕ ТРЕБОВАНИЯ**
- Виртуальная машина с Ubuntu 24.04
- Доступ с правами root или sudo
- Базовые знания команд Linux

---

## **🛠️ ЧАСТЬ 1: УСТАНОВКА И БАЗОВАЯ НАСТРОЙКА UFW**

### **Шаг 1.1: Проверка состояния фаервола**

```bash
# Проверить статус UFW
sudo ufw status verbose

# Если UFW не установлен
sudo apt update
sudo apt install ufw
```

### **Шаг 1.2: Базовая настройка политик по умолчанию**

```bash
# Запретить все входящие соединения
sudo ufw default deny incoming

# Разрешить все исходящие соединения
sudo ufw default allow outgoing

# Просмотреть правила
sudo ufw show added
```

### **Шаг 1.3: Разрешение базовых служб**

```bash
# Разрешить SSH (ОБЯЗАТЕЛЬНО перед включением!)
sudo ufw allow ssh
# или по порту
sudo ufw allow 22/tcp

# Разрешить HTTP и HTTPS
sudo ufw allow http
sudo ufw allow https

# Разрешить DNS
sudo ufw allow 53/udp

# Разрешить ping (ICMP)
sudo ufw allow in proto icmp
```

---

## **🔧 ЧАСТЬ 2: РАСШИРЕННАЯ НАСТРОЙКА UFW**

### **Шаг 2.1: Создание правил для конкретных сетей**

```bash
# Разрешить SSH только из определенной подсети
sudo ufw allow from 192.168.1.0/24 to any port 22

# Разрешить доступ к веб-серверу только из определенного IP
sudo ufw allow from 192.168.1.100 to any port 80,443

# Запретить доступ из конкретной подсети
sudo ufw deny from 10.0.0.0/8 to any
```

### **Шаг 2.2: Настройка правил для конкретных интерфейсов**

```bash
# Узнать имена интерфейсов
ip addr show

# Разрешить доступ только на определенном интерфейсе
sudo ufw allow in on eth0 to any port 80
sudo ufw allow in on eth1 to any port 22
```

### **Шаг 2.3: Создание комплексных правил**

```bash
# Разрешить доступ к MySQL только из внутренней сети
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Разрешить диапазон портов для приложения
sudo ufw allow 8000:8010/tcp

# Лимит подключений для защиты от брут-форса
sudo ufw limit ssh
```

---

## **📊 ЧАСТЬ 3: ПРАКТИЧЕСКИЕ СЦЕНАРИИ**

### **Сценарий 3.1: Веб-сервер**

```bash
#!/bin/bash
# Настройка фаервола для веб-сервера

# Сброс всех правил
sudo ufw --force reset

# Политики по умолчанию
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Базовые службы
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Дополнительные порты для веб-приложений
sudo ufw allow 8080/tcp
sudo ufw allow 8443/tcp

# Включение фаервола
sudo ufw enable
```

### **Сценарий 3.2: Сервер базы данных**

```bash
#!/bin/bash
# Настройка фаервола для MySQL сервера

sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH только из внутренней сети
sudo ufw allow from 192.168.1.0/24 to any port 22

# MySQL только из внутренней сети
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Мониторинг только с localhost
sudo ufw allow from 127.0.0.1 to any port 9090

sudo ufw enable
```

---

## **⚡ ЧАСТЬ 4: РАБОТА С IPTABLES (ПРОДВИНУТЫЙ УРОВЕНЬ)**

### **Шаг 4.1: Создание bash-скрипта для iptables**

```bash
sudo nano /usr/local/bin/firewall-setup.sh
```

```bash
#!/bin/bash
# Скрипт настройки iptables для Ubuntu 24.04

# Сброс всех правил
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Политики по умолчанию
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Разрешить loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Разрешить установленные и связанные соединения
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Разрешить SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Разрешить ping
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Логирование отклоненных пакетов
iptables -A INPUT -j LOG --log-prefix "IPTABLES_DENIED: " --log-level 4

echo "Правила iptables применены"
```

### **Шаг 4.2: Сделать скрипт исполняемым и запустить**

```bash
sudo chmod +x /usr/local/bin/firewall-setup.sh
sudo /usr/local/bin/firewall-setup.sh
```

### **Шаг 4.3: Сохранение правил iptables**

```bash
# Установка утилиты для сохранения правил
sudo apt install iptables-persistent

# Сохранить текущие правила
sudo netfilter-persistent save

# Проверить сохраненные правила
sudo iptables-save
```

---

## **🔍 ЧАСТЬ 5: МОНИТОРИНГ И ДИАГНОСТИКА**

### **Шаг 5.1: Команды мониторинга UFW**

```bash
# Просмотр статуса
sudo ufw status verbose

# Просмотр нумерованных правил
sudo ufw status numbered

# Просмотр логов
sudo tail -f /var/log/ufw.log

# Просмотр правил в детальном формате
sudo ufw show raw
```

### **Шаг 5.2: Команды мониторинга iptables**

```bash
# Просмотр всех правил
sudo iptables -L -v -n

# Просмотр правил с номерами
sudo iptables -L -v -n --line-numbers

# Просмотр таблицы NAT
sudo iptables -t nat -L -v -n

# Мониторинг трафика
sudo iptables -L -v -n | head -20
```

### **Шаг 5.3: Тестирование правил**

```bash
# Проверка доступности порта с другой машины
telnet ваш_сервер 22
telnet ваш_сервер 80

# Сканирование портов (с другой машины)
nmap -sS ваш_сервер

# Проверка ping
ping ваш_сервер
```

---

## **🛡️ ЧАСТЬ 6: БЕЗОПАСНОСТЬ И ОПТИМИЗАЦИЯ**

### **Шаг 6.1: Защита от частых атак**

```bash
# Ограничение количества подключений к SSH
sudo ufw limit ssh

# Защита от ping-флуда
sudo ufw limit proto icmp

# Блокировка подозрительных IP
sudo ufw deny from 123.456.789.123
```

### **Шаг 6.2: Создание скрипта автоматической блокировки**

```bash
sudo nano /usr/local/bin/block-suspicious.sh
```

```bash
#!/bin/bash
# Скрипт автоматической блокировки подозрительных IP

LOGFILE="/var/log/ufw.log"
BLOCK_THRESHOLD=5

# Анализ логов и блокировка IP с множеством failed соединений
tail -1000 $LOGFILE | grep "UFW BLOCK" | awk '{print $12}' | \
sort | uniq -c | sort -nr | while read count ip; do
    if [ $count -gt $BLOCK_THRESHOLD ]; then
        echo "Блокируем IP $ip за $count попыток"
        ufw deny from $ip
    fi
done
```

---

## **📝 ЧАСТЬ 7: ЗАДАНИЯ ДЛЯ САМОСТОЯТЕЛЬНОГО ВЫПОЛНЕНИЯ**

### **Задание 7.1: Базовая настройка**
1. Установите и настройте UFW с политиками:
   - Запретить все входящие
   - Разрешить все исходящие
   - Разрешить SSH, HTTP, HTTPS
   - Включите фаервол

### **Задание 7.2: Расширенная настройка**
1. Создайте правила для:
   - Разрешить доступ к порту 3306 только из сети 192.168.1.0/24
   - Запретить доступ с IP 10.0.0.100
   - Разрешить диапазон портов 8000-8010

### **Задание 7.3: Сценарий безопасности**
1. Настройте фаервол для сервера с:
   - Веб-сервер (порты 80, 443)
   - База данных (порт 3306) - только для внутренней сети
   - SSH - только для администраторов
   - Запретить все остальное

### **Задание 7.4: Мониторинг и диагностика**
1. Проверьте работу правил:
   - Просканируйте порты с другой машины
   - Проверьте логи фаервола
   - Протестируйте доступ к службам

---

## **🎯 КРИТЕРИИ ОЦЕНКИ**

| Задание | Максимальный балл | Критерии оценки |
|---------|------------------|-----------------|
| 7.1 | 3 балла | Правильная установка и базовая настройка |
| 7.2 | 3 балла | Корректное создание расширенных правил |
| 7.3 | 2 балла | Адекватная конфигурация для сценария |
| 7.4 | 2 балла | Умение диагностировать и мониторить |

**Шкала оценки:**
- **9-10 баллов:** Отлично
- **7-8 баллов:** Хорошо  
- **5-6 баллов:** Удовлетворительно
- **0-4 балла:** Неудовлетворительно

---

## **⚠️ ВАЖНЫЕ ПРЕДУПРЕЖДЕНИЯ**

1. **Всегда разрешайте SSH до включения фаервола!**
2. **Тестируйте правила перед применением в production**
3. **Сохраняйте резервные копии конфигураций**
4. **Имейте план аварийного доступа (KVM/IPMI)**

## **💡 ПОЛЕЗНЫЕ КОМАНДЫ ДЛЯ ЭКСТРЕННЫХ СИТУАЦИЙ**

```bash
# Отключить фаервол (если заблокировали себя)
sudo ufw disable

# Сбросить все правила UFW
sudo ufw --force reset

# Экстренный сброс iptables
sudo iptables -F && sudo iptables -X
```

---

# **🔧 САМОСТОЯТЕЛЬНАЯ РАБОТА: НАСТРОЙКА МЕЖСЕТЕВОГО ЭКРАНА НА KALI LINUX**

## **🎯 ЦЕЛЬ РАБОТЫ**
Научиться настраивать и управлять межсетевым экраном на Kali Linux с использованием `iptables`, `nftables` и дополнительных инструментов безопасности.

---

## **📋 ПРЕДВАРИТЕЛЬНЫЕ ТРЕБОВАНИЯ**
- Установленная Kali Linux 2024.x
- Доступ с правами root
- Базовые знания сетевых технологий
- Понимание принципов пентестинга

---

## **🛠️ ЧАСТЬ 1: НАСТРОЙКА БАЗОВОГО ФАЕРВОЛА С IPTABLES**

### **Шаг 1.1: Проверка текущих правил iptables**

```bash
# Проверить текущие правила
iptables -L -v -n

# Проверить правила NAT
iptables -t nat -L -v -n

# Посмотреть все таблицы
iptables-save
```

### **Шаг 1.2: Создание базового скрипта безопасности**

```bash
# Создаем скрипт настройки фаервола
nano /root/firewall-setup.sh
```

```bash
#!/bin/bash
# Kali Linux Firewall Setup Script
# Автор: [Ваше Имя]
# Дата: $(date)

echo "[+] Начинаем настройку фаервола Kali Linux"

# ==================== СБРОС ПРАВИЛ ====================
echo "[+] Сбрасываем все правила iptables"
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# ==================== ПОЛИТИКИ ПО УМОЛЧАНИЮ ====================
echo "[+] Устанавливаем политики по умолчанию"
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# ==================== РАЗРЕШЕНИЕ LOOPBACK ====================
echo "[+] Разрешаем loopback интерфейс"
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# ==================== РАЗРЕШЕНИЕ УСТАНОВЛЕННЫХ СОЕДИНЕНИЙ ====================
echo "[+] Разрешаем установленные и связанные соединения"
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# ==================== ОСНОВНЫЕ СЛУЖБЫ ====================
echo "[+] Настраиваем правила для основных служб"

# SSH - ограничиваем попытки подключения
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m limit --limit 3/min --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j DROP

# Веб-сервисы для инструментов пентестинга
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # HTTPS
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT  # Альтернативный веб-порт

# ==================== ИНСТРУМЕНТЫ ПЕНТЕСТИНГА ====================
echo "[+] Настраиваем порты для инструментов пентестинга"

# Metasploit
iptables -A INPUT -p tcp --dport 4444 -j ACCEPT
iptables -A INPUT -p tcp --dport 55553 -j ACCEPT

# Burp Suite
iptables -A INPUT -p tcp --dport 8081 -j ACCEPT

# Сканеры и фреймворки
iptables -A INPUT -p tcp --dport 8834 -j ACCEPT   # Nessus

# ==================== ЗАЩИТА ОТ АТАК ====================
echo "[+] Настраиваем защиту от распространенных атак"

# Защита от ping-флуда
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Защита от port scanning
iptables -N PORTSCAN
iptables -A PORTSCAN -m limit --limit 1/hour --limit-burst 3 -j LOG --log-prefix "Portscan detected: "
iptables -A PORTSCAN -j DROP

# Блокировка NULL пакетов
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# Блокировка XMAS пакетов
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# ==================== ЛОГИРОВАНИЕ ====================
echo "[+] Настраиваем логирование"

# Логирование отклоненных пакетов
iptables -A INPUT -j LOG --log-prefix "IPTABLES_INPUT_DENIED: " --log-level 4
iptables -A FORWARD -j LOG --log-prefix "IPTABLES_FORWARD_DENIED: " --log-level 4

# Ограничение логов чтобы не засорять диск
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables: "

echo "[+] Настройка фаервола завершена"
echo "[+] Текущие правила:"
iptables -L -v -n
```

### **Шаг 1.3: Запуск и проверка скрипта**

```bash
# Делаем скрипт исполняемым
chmod +x /root/firewall-setup.sh

# Запускаем скрипт
/root/firewall-setup.sh

# Проверяем правила
iptables -L -v -n --line-numbers
```

---

## **🔧 ЧАСТЬ 2: НАСТРОЙКА NFTABLES (СОВРЕМЕННАЯ АЛЬТЕРНАТИВА)**

### **Шаг 2.1: Установка и базовая настройка nftables**

```bash
# Установка nftables (обычно уже установлен в Kali)
apt update && apt install nftables -y

# Запуск службы
systemctl enable nftables
systemctl start nftables
```

### **Шаг 2.2: Создание конфигурации nftables**

```bash
# Создаем конфигурационный файл
nano /etc/nftables.conf
```

```bash
#!/usr/sbin/nft -f

# Kali Linux NFTables Configuration
# Flush all rules
flush ruleset

# Define variables
define admin_net = { 192.168.1.0/24 }
define metasploit_ports = { 4444, 55553 }
define web_ports = { 80, 443, 8080, 8081 }

table inet firewall {
    
    # Set definitions
    set blacklist {
        type ipv4_addr
        flags timeout
    }
    
    # Base chain for input
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Allow loopback
        iifname "lo" accept
        iifname != "lo" ip daddr 127.0.0.0/8 drop
        
        # Allow established/related connections
        ct state established,related accept
        
        # Allow invalid for some pentesting tools
        ct state invalid limit rate 5/second log prefix "nftables invalid: " drop
        
        # ICMP (ping)
        ip protocol icmp icmp type echo-request limit rate 1/second accept
        ip protocol icmp icmp type echo-request drop
        
        # SSH with rate limiting
        tcp dport 22 ct state new limit rate 3/minute accept
        
        # Web services
        tcp dport $web_ports accept
        
        # Pentesting tools
        tcp dport $metasploit_ports accept
        tcp dport 8834 accept  # Nessus
        
        # Log and drop everything else
        log prefix "nftables denied: " group 0 drop
    }
    
    # Forward chain
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    # Output chain
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

### **Шаг 2.3: Применение конфигурации nftables**

```bash
# Применяем конфигурацию
nft -f /etc/nftables.conf

# Проверяем правила
nft list ruleset

# Сохраняем правила для автозагрузки
systemctl enable nftables
systemctl restart nftables
```

---

## **📊 ЧАСТЬ 3: ПРАКТИЧЕСКИЕ СЦЕНАРИИ ДЛЯ ПЕНТЕСТИНГА**

### **Сценарий 3.1: Режим сканирования сети**

```bash
nano /root/firewall-scanning.sh
```

```bash
#!/bin/bash
# Firewall configuration for network scanning

echo "[+] Setting up firewall for network scanning"

# Reset rules
iptables -F
iptables -X

# Default policies
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# Allow all traffic for scanning
echo "[+] Firewall configured for permissive scanning mode"
iptables -L -v -n
```

### **Сценарий 3.2: Режим стелс-прослушивания**

```bash
nano /root/firewall-stealth.sh
```

```bash
#!/bin/bash
# Stealth firewall configuration

echo "[+] Setting up stealth firewall"

# Reset all rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Default policies - DROP everything
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow DNS for reconnaissance
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p udp --sport 53 -j ACCEPT

# Allow specific outbound connections for tools
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT    # HTTP
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT   # HTTPS
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT    # SSH

# Drop and log everything else
iptables -A INPUT -j LOG --log-prefix "STEALTH_INPUT_DROP: "
iptables -A OUTPUT -j LOG --log-prefix "STEALTH_OUTPUT_DROP: "

echo "[+] Stealth firewall configured"
iptables -L -v -n
```

---

## **⚡ ЧАСТЬ 4: РАСШИРЕННЫЕ ВОЗМОЖНОСТИ БЕЗОПАСНОСТИ**

### **Шаг 4.1: Настройка защиты от DDoS атак**

```bash
nano /root/ddos-protection.sh
```

```bash
#!/bin/bash
# DDoS Protection Script

echo "[+] Configuring DDoS protection"

# Create custom chains for DDoS protection
iptables -N SYN_FLOOD
iptables -N PORTSCAN
iptables -N ICMP_FLOOD

# SYN Flood protection
iptables -A SYN_FLOOD -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j RETURN
iptables -A SYN_FLOOD -j DROP
iptables -A INPUT -p tcp --syn -j SYN_FLOOD

# Portscan protection
iptables -A PORTSCAN -m recent --name portscan --rcheck --seconds 60 --hitcount 5 -j DROP
iptables -A PORTSCAN -m recent --name portscan --set -j DROP
iptables -A INPUT -p tcp --tcp-flags FIN,URG,PSH FIN,URG,PSH -j PORTSCAN

# ICMP Flood protection
iptables -A ICMP_FLOOD -p icmp --icmp-type echo-request -m limit --limit 1/s -j RETURN
iptables -A ICMP_FLOOD -j DROP
iptables -A INPUT -p icmp --icmp-type echo-request -j ICMP_FLOOD

echo "[+] DDoS protection configured"
```

### **Шаг 4.2: Настройка гео-фильтрации**
Вариант 1
```
#!/bin/bash
# Альтернативный способ блокировки по странам без xt_geoip

# Устанавливаем ipset
sudo apt install -y ipset

# Создаем наборы IP для стран
sudo ipset create blocked_countries hash:net

# Добавляем подсети конкретных стран (пример для России и Китая)
# Эти данные нужно получить из внешних источников
sudo ipset add blocked_countries 192.168.1.0/24  # пример
# sudo ipset add blocked_countries x.x.x.x/x      # реальные подсети

# Применяем правила iptables
sudo iptables -I INPUT -m set --match-set blocked_countries src -j DROP
sudo iptables -I FORWARD -m set --match-set blocked_countries src -j DROP

# Сохраняем правила
sudo ipset save blocked_countries > /etc/ipset.conf
```
Вариант 2

```bash
# Установка geoip базы
apt install xtables-addons-dkms libtext-csv-xs-perl -y

# Скачивание geoip базы
mkdir -p /usr/share/xt_geoip
/usr/lib/xtables-addons/xt_geoip_dl
/usr/lib/xtables-addons/xt_geoip_build -D /usr/share/xt_geoip *.csv

# Правила гео-фильтрации
iptables -A INPUT -m geoip --src-cc RU,CN -j DROP
iptables -A INPUT -m geoip --src-cc US,CA,GB -j ACCEPT
```

---

## **🔍 ЧАСТЬ 5: МОНИТОРИНГ И АНАЛИЗ УГРОЗ**

### **Шаг 5.1: Настройка расширенного логирования**

```bash
# Создаем скрипт для анализа логов
nano /root/firewall-monitor.sh
```

```bash
#!/bin/bash
# Firewall Monitoring Script

LOG_FILE="/var/log/kern.log"
ALERT_THRESHOLD=10

echo "[+] Starting firewall monitoring"

# Monitor for portscan attempts
tail -f $LOG_FILE | while read line; do
    # Detect portscans
    if echo "$line" | grep -q "Portscan detected"; then
        echo "ALERT: Portscan detected - $line"
    fi
    
    # Detect SYN floods
    if echo "$line" | grep -q "SYN_FLOOD"; then
        echo "ALERT: SYN flood detected - $line"
    fi
    
    # Count denied packets per IP
    denied_ip=$(echo "$line" | grep "IPTABLES_DENIED" | awk '{print $10}')
    if [ ! -z "$denied_ip" ]; then
        count=$(grep "IPTABLES_DENIED.*$denied_ip" $LOG_FILE | wc -l)
        if [ $count -gt $ALERT_THRESHOLD ]; then
            echo "ALERT: IP $denied_ip has $count denied packets"
        fi
    fi
done
```

### **Шаг 5.2: Инструменты мониторинга в реальном времени**

```bash
# Установка дополнительных инструментов мониторинга
apt install nethogs iftop iptraf-ng -y

# Запуск мониторинга
iftop -i eth0        # Просмотр сетевого трафика
nethogs eth0         # Мониторинг трафика по процессам
iptraf-ng            # Комплексный сетевой мониторинг
```

---

## **📝 ЧАСТЬ 6: ЗАДАНИЯ ДЛЯ САМОСТОЯТЕЛЬНОГО ВЫПОЛНЕНИЯ**

### **Задание 6.1: Базовая настройка безопасности**
1. Настройте iptables с политикой "deny all" на вход
2. Разрешите только SSH с лимитом 3 подключения в минуту
3. Разрешите веб-порты (80, 443, 8080)
4. Настройте правила для инструментов пентестинга

### **Задание 6.2: Сценарий пентест-лаборатории**
1. Создайте конфигурацию фаервола для:
   - Веб-приложения на порту 8080
   - Базы данных на порту 3306 (только localhost)
   - Metasploit на портах 4444, 55553
   - Заблокируйте все остальные порты

### **Задание 6.3: Защита от атак**
1. Реализуйте защиту от:
   - SYN flood атак
   - Port scanning
   - Ping flood
   - Добавьте логирование всех блокировок

### **Задание 6.4: Мониторинг и ответ на инциденты**
1. Настройте систему мониторинга:
   - Обнаружение порт-сканирования
   - Анализ подозрительного трафика
   - Автоматическое блокирование IP при превышении лимита

---

## **🎯 КРИТЕРИИ ОЦЕНКИ**

| Задание | Максимальный балл | Критерии оценки |
|---------|------------------|-----------------|
| 6.1 | 3 балла | Корректная базовая настройка iptables |
| 6.2 | 3 балла | Адекватная конфигурация для сценария |
| 6.3 | 2 балла | Эффективная защита от атак |
| 6.4 | 2 балла | Функциональная система мониторинга |

**Шкала оценки:**
- **9-10 баллов:** Отлично - полное понимание темы
- **7-8 баллов:** Хорошо - небольшие ошибки в настройке
- **5-6 баллов:** Удовлетворительно - основные правила работают
- **0-4 балла:** Неудовлетворительно - требуется доработка

---

## **⚠️ ВАЖНЫЕ ПРЕДУПРЕЖДЕНИЯ ДЛЯ KALI LINUX**

### **Особенности безопасности Kali:**
1. **Kali по умолчанию работает под root** - будьте осторожны с правилами
2. **Многие инструменты пентестинга требуют специфичных портов**
3. **Тестируйте правила на виртуальной машине перед применением**

### **Экстренные команды:**
```bash
# Экстренный сброс всех правил (если заблокировали себя)
iptables -F && iptables -X && iptables -P INPUT ACCEPT

# Временное отключение фаервола
systemctl stop nftables
systemctl stop iptables

# Разрешить весь трафик (опасно!)
iptables -P INPUT ACCEPT && iptables -P OUTPUT ACCEPT
```

### **Сохранение правил для автозагрузки:**
```bash
# Для iptables
apt install iptables-persistent
netfilter-persistent save

# Для nftables
systemctl enable nftables
nft list ruleset > /etc/nftables.conf
```

---

## **💡 ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ**

### **Полезные команды для отладки:**
```bash
# Просмотр правил с номерами
iptables -L -n --line-numbers

# Удаление правила по номеру
iptables -D INPUT 3

# Мониторинг в реальном времени
watch -n 1 'iptables -L -v -n'

# Проверка конкретного порта
nc -zv localhost 22
```

### **Документация для дальнейшего изучения:**
- `man iptables`
- `man nft`
- `man ufw`
- Kali Linux Documentation

**Успешного выполнения самостоятельной работы на Kali Linux! 🚀💀**