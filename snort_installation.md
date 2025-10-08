
---

# **🔍 ПОЛНАЯ ИНСТРУКЦИЯ ПО УСТАНОВКЕ И НАСТРОЙКЕ SNORT НА KALI LINUX**

## **🎯 ЦЕЛЬ**
Установить и настроить Snort - мощную систему обнаружения вторжений (IDS/IPS) на Kali Linux для мониторинга сетевой активности.

---

## **📋 ПРЕДВАРИТЕЛЬНЫЕ ТРЕБОВАНИЯ**
- Kali Linux 2024.x (обновленная)
- Минимум 2 ГБ свободного места
- Сетевой интерфейс в режиме promiscuous
- Права root

---

## **🛠️ ЧАСТЬ 1: ПОДГОТОВКА СИСТЕМЫ**

### **Шаг 1.1: Обновление системы**
```bash
# Обновляем список пакетов
sudo apt update && sudo apt upgrade -y

# Устанавливаем необходимые зависимости
sudo apt install -y build-essential libpcap-dev libpcre3-dev libdumbnet-dev \
bison flex zlib1g-dev liblzma-dev openssl libssl-dev libnghttp2-dev \
autotools-dev libdbi-perl libtool
```

### **Шаг 1.2: Установка дополнительных компонентов**
```bash
# Установка инструментов для компиляции
sudo apt install -y cmake cpputest libsqlite3-dev libtool uuid-dev \
libuv1-dev libcmocka-dev libunwind-dev libmnl-dev

# Установка Python компонентов (для правил и управления)
sudo apt install -y python3-pip python3-dev
pip3 install --upgrade pip
```

---

## **📥 ЧАСТЬ 2: УСТАНОВКА SNORT**

### **Шаг 2.1: Создание рабочей директории**
```bash
# Создаем директорию для установки
sudo mkdir -p /opt/snort_src
cd /opt/snort_src
```

### **Шаг 2.2: Установка DAQ (Data Acquisition Library)**
```bash
# Скачиваем и распаковываем DAQ
wget https://www.snort.org/downloads/snort/daq-2.0.10.tar.gz
tar -xzvf daq-2.0.10.tar.gz
cd daq-2.0.10

# Компилируем и устанавливаем
./configure && make && sudo make install

# Возвращаемся в исходную директорию
cd ..
```

### **Шаг 2.3: Установка Snort**
```bash
# Скачиваем Snort
wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
tar -xzvf snort-2.9.20.tar.gz
cd snort-2.9.20

# Конфигурируем с поддержкой всех функций
./configure --enable-sourcefire --disable-open-appid

# Компилируем и устанавливаем
make && sudo make install

# Обновляем библиотеки
sudo ldconfig
```

### **Шаг 2.4: Проверка установки**
```bash
# Проверяем версию Snort
snort --version

# Должны увидеть примерно:
# ,,_     -*> Snort! <*-
# o"  )~   Version 2.9.20 GRE (Build 1)
# ''''    By Martin Roesch & The Snort Team
```

---

## **⚙️ ЧАСТЬ 3: БАЗОВАЯ КОНФИГУРАЦИЯ**

### **Шаг 3.1: Создание структуры директорий**
```bash
# Создаем системные директории
sudo mkdir -p /etc/snort
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /etc/snort/preproc_rules
sudo mkdir -p /etc/snort/so_rules
sudo mkdir -p /var/log/snort
sudo mkdir -p /usr/local/lib/snort_dynamicrules

# Создаем дополнительные директории
sudo mkdir -p /etc/snort/rules/iplists
sudo mkdir -p /etc/snort/rules/whitelists
sudo mkdir -p /etc/snort/rules/blacklists
```

### **Шаг 3.2: Копирование конфигурационных файлов**
```bash
# Копируем конфигурационные файлы из исходников
cd /opt/snort_src/snort-2.9.20
sudo cp etc/*.conf* /etc/snort/
sudo cp etc/*.map /etc/snort/

# Копируем конфигурацию по умолчанию
sudo cp snort.conf /etc/snort/
```

### **Шаг 3.3: Настройка прав доступа**
```bash
# Устанавливаем правильные права
sudo chmod -R 5775 /etc/snort/
sudo chmod -R 5775 /var/log/snort/
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
sudo chown -R root:root /etc/snort/
```

---

## **🔧 ЧАСТЬ 4: НАСТРОЙКА CONFIG ФАЙЛА**

### **Шаг 4.1: Редактирование основного конфига**
```bash
# Открываем конфигурационный файл
sudo nano /etc/snort/snort.conf
```

### **Шаг 4.2: Основные настройки (найти и изменить)**
```bash
# Устанавливаем сетевые переменные (пример для домашней сети)
var HOME_NET 192.168.1.0/24
var EXTERNAL_NET !$HOME_NET

# Указываем серверы
var DNS_SERVERS $HOME_NET
var SMTP_SERVERS $HOME_NET
var HTTP_SERVERS $HOME_NET
var SQL_SERVERS $HOME_NET
var TELNET_SERVERS $HOME_NET

# Настраиваем порты
var HTTP_PORTS 80,443,8080,8443
var SHELLCODE_PORTS !80
var ORACLE_PORTS 1521

# Пути к правилам
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
var WHITE_LIST_PATH /etc/snort/rules/whitelists
var BLACK_LIST_PATH /etc/snort/rules/blacklists
```

### **Шаг 4.3: Настройка правил**
```bash
# В файле /etc/snort/snort.conf находим и раскомментируем:
include $RULE_PATH/local.rules
include $RULE_PATH/icmp.rules

# Включаем предпроцессоры
preprocessor normalize_ip4
preprocessor normalize_tcp: ips ecn stream
preprocessor normalize_icmp4
preprocessor normalize_ip6
preprocessor frag3_global: max_frags 65536
preprocessor frag3_engine: policy windows detect_anomalies
preprocessor stream5_global: max_tcp 8192, track_tcp yes, track_udp yes
preprocessor stream5_tcp: policy windows, detect_anomalies
preprocessor http_inspect
preprocessor rpc_decode
preprocessor performance
```

---

## **📋 ЧАСТЬ 5: СОЗДАНИЕ ПРАВИЛ**

### **Шаг 5.1: Создание локальных правил**
```bash
# Создаем файл локальных правил
sudo nano /etc/snort/rules/local.rules
```

### **Шаг 5.2: Добавление тестовых правил**
```bash
# Добавляем в файл local.rules следующие правила:

# Правило для обнаружения ping (ICMP)
alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)

# Правило для обнаружения SSH сканирования
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"SSH Connection Attempt"; flags:S; sid:1000002; rev:1;)

# Правило для обнаружения веб-атак
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS (msg:"Web Attack Attempt"; content:"/etc/passwd"; nocase; sid:1000003; rev:1;)

# Правило для обнаружения SQL инъекций
alert tcp $EXTERNAL_NET any -> $SQL_SERVERS 1433 (msg:"SQL Injection Attempt"; content:"union"; nocase; sid:1000004; rev:1;)

# Правило для обнаружения порт-сканирования
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"Port Scan Attempt"; flags:S; detection_filter: track by_src, count 5, seconds 10; sid:1000005; rev:1;)
```

### **Шаг 5.3: Создание белых и черных списков**
```bash
# Создаем белый список (доверенные IP)
sudo nano /etc/snort/rules/whitelists/whitelist.rules

# Добавляем доверенные IP (пример):
# 192.168.1.100 - ваш доверенный компьютер
```

```bash
# Создаем черный список (блокируемые IP)
sudo nano /etc/snort/rules/blacklists/blacklist.rules

# Добавляем подозрительные IP (пример):
# 10.0.0.100 - подозрительный IP
```

---

## **🎯 ЧАСТЬ 6: ТЕСТИРОВАНИЕ SNORT**

### **Шаг 6.1: Проверка конфигурации**
```bash
# Тестируем конфигурационный файл
sudo snort -T -c /etc/snort/snort.conf -i eth0

# Успешный вывод должен содержать:
# Snort successfully validated the configuration!
# Snort exiting
```

### **Шаг 6.2: Запуск в режиме IDS**
```bash
# Запускаем Snort в фоновом режиме
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0 -D

# Или с логированием в файл
sudo snort -c /etc/snort/snort.conf -i eth0 -l /var/log/snort -K ascii
```

### **Шаг 6.3: Тестирование работы**
```bash
# С другой машины выполните ping до Kali
ping [IP_ADDRESS_KALI]

# Или попробуйте подключиться по SSH
ssh [USER]@[IP_ADDRESS_KALI]

# Проверьте логи Snort
sudo tail -f /var/log/snort/alert
```

---

## **📊 ЧАСТЬ 7: НАСТРОЙКА РАСШИРЕННЫХ ВОЗМОЖНОСТЕЙ**

### **Шаг 7.1: Установка PulledPork для автоматического обновления правил**
```bash
# Устанавливаем зависимости
sudo apt install -y libcrypt-ssleay-perl liblwp-protocol-https-perl

# Клонируем PulledPork
cd /opt
sudo git clone https://github.com/shirkdog/pulledpork3.git
cd pulledpork3

# Копируем конфигурационный файл
sudo cp pulledpork.conf /etc/snort/
```

### **Шаг 7.2: Настройка PulledPork**
```bash
# Редактируем конфиг
sudo nano /etc/snort/pulledpork.conf

# Основные настройки:
rule_url=https://www.snort.org/rules/community|community.rules|$oinkcode
rule_url=https://rules.emergingthreats.net/$openappid/$emerging_proof|emerging.rules.tar.gz|
rule_path=/etc/snort/rules/snort.rules
local_rules=/etc/snort/rules/local.rules
sid_msg=/etc/snort/sid-msg.map
sid_msg_version=2
```

### **Шаг 7.3: Запуск автоматического обновления правил**
```bash
# Тестовый запуск
sudo perl pulledpork.pl -c /etc/snort/pulledpork.conf -T

# Реальное обновление
sudo perl pulledpork.pl -c /etc/snort/pulledpork.conf
```

---

## **🔧 ЧАСТЬ 8: СОЗДАНИЕ СЕРВИСА SYSTEMD**

### **Шаг 8.1: Создание сервисного файла**
```bash
# Создаем файл сервиса
sudo nano /etc/systemd/system/snort.service
```

### **Шаг 8.2: Конфигурация сервиса**
```bash
[Unit]
Description=Snort NIDS Daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -q -c /etc/snort/snort.conf -i eth0 -D
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

### **Шаг 8.3: Запуск сервиса**
```bash
# Перезагружаем systemd
sudo systemctl daemon-reload

# Включаем автозагрузку
sudo systemctl enable snort

# Запускаем сервис
sudo systemctl start snort

# Проверяем статус
sudo systemctl status snort
```

---

## **📈 ЧАСТЬ 9: МОНИТОРИНГ И АНАЛИЗ**

### **Шаг 9.1: Просмотр логов в реальном времени**
```bash
# Мониторинг логов
sudo tail -f /var/log/snort/alert

# Просмотр статистики
sudo snort -c /etc/snort/snort.conf -i eth0 --dump-plugin-stats

# Анализ производительности
sudo snort -c /etc/snort/snort.conf -i eth0 --perfmon-file snort_perf.log
```

### **Шаг 9.2: Использование Barnyard2 для вывода в базу данных**
```bash
# Установка Barnyard2
cd /opt/snort_src
sudo git clone https://github.com/firnsy/barnyard2.git
cd barnyard2
./autogen.sh
./configure --with-mysql
make && sudo make install

# Настройка подключения к БД
sudo cp etc/barnyard2.conf /etc/snort/
```

---

## **🛡️ ЧАСТЬ 10: ОПТИМИЗАЦИЯ И БЕЗОПАСНОСТЬ**

### **Шаг 10.1: Создание скрипта управления**
```bash
# Создаем скрипт управления Snort
sudo nano /usr/local/bin/snort-manager.sh
```

```bash
#!/bin/bash
# Snort Management Script

case "$1" in
    start)
        echo "Starting Snort..."
        systemctl start snort
        ;;
    stop)
        echo "Stopping Snort..."
        systemctl stop snort
        ;;
    restart)
        echo "Restarting Snort..."
        systemctl restart snort
        ;;
    status)
        systemctl status snort
        ;;
    logs)
        tail -f /var/log/snort/alert
        ;;
    update)
        echo "Updating rules..."
        cd /opt/pulledpork3
        perl pulledpork.pl -c /etc/snort/pulledpork.conf
        systemctl restart snort
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status|logs|update}"
        exit 1
        ;;
esac
```

```bash
# Делаем скрипт исполняемым
sudo chmod +x /usr/local/bin/snort-manager.sh
```

### **Шаг 10.2: Настройка регулярных задач**
```bash
# Создаем cron задание для обновления правил
sudo crontab -e

# Добавляем строку (обновление каждый день в 2:00):
0 2 * * * /usr/local/bin/snort-manager.sh update

# Очистка логов раз в неделю
0 3 * * 0 find /var/log/snort -name "*.log" -mtime +7 -delete
```

---

## **🎯 ЧАСТЬ 11: ТЕСТИРОВАНИЕ И ВАЛИДАЦИЯ**

### **Тест 1: Проверка обнаружения ICMP**
```bash
# С другой машины выполните:
ping [IP_KALI]

# В логах Snort должна появиться запись:
# ICMP Ping Detected
```

### **Тест 2: Проверка обнаружения порт-сканирования**
```bash
# С другой машины выполните сканирование:
nmap -sS [IP_KALI]

# В логах должна появиться запись:
# Port Scan Attempt
```

### **Тест 3: Проверка веб-атак**
```bash
# Попробуйте выполнить HTTP запрос с подозрительным содержанием
curl http://[IP_KALI]/etc/passwd

# Должна сработать сигнатура веб-атаки
```

---

## **⚠️ РЕШЕНИЕ ЧАСТЫХ ПРОБЛЕМ**

### **Проблема 1: "snort: error while loading shared libraries"**
```bash
# Решение: обновить кэш библиотек
sudo ldconfig
```

### **Проблема 2: "ERROR: Can't set DAQ type to pcap"**
```bash
# Решение: проверить сетевой интерфейс
ip link show
sudo snort -c /etc/snort/snort.conf -i [CORRECT_INTERFACE]
```

### **Проблема 3: "ERROR: /etc/snort/snort.conf(0) Unable to open rules file"**
```bash
# Решение: проверить пути и права доступа
sudo chmod -R 5775 /etc/snort/
sudo ls -la /etc/snort/rules/
```

### **Проблема 4: Низкая производительность**
```bash
# Решение: оптимизировать правила
# В snort.conf добавить:
config detection: search-method ac-split
config profile_rules: print_all
```

---

## **💡 ПОЛЕЗНЫЕ КОМАНДЫ ДЛЯ ПОВСЕДНЕВНОГО ИСПОЛЬЗОВАНИЯ**

```bash
# Быстрый запуск для тестирования
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

# Запуск с подробным логированием
sudo snort -c /etc/snort/snort.conf -i eth0 -l /var/log/snort -K ascii

# Проверка конкретного правила
sudo snort -c /etc/snort/snort.conf -i eth0 -A alert_fast

# Мониторинг в реальном времени
sudo tail -f /var/log/snort/alert | grep -E "(ICMP|Port Scan|SQL)"
```

**Теперь у вас полностью настроенная и работающая система обнаружения вторжений Snort на Kali Linux! 🚀🔍**

Система готова к мониторингу сетевой активности и обнаружению потенциальных угроз безопасности.

# Ubuntu 22.04
# 🚀 Установка и настройка Snort на Ubuntu 22.04: Полное руководство

## 📋 Предварительные требования

### **1. Обновление системы**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### **2. Установка зависимостей**
```bash
sudo apt install -y build-essential libpcap-dev libpcre3-dev libnet1-dev \
libyaml-0-2 libyaml-dev zlib1g zlib1g-dev libcap-ng-dev libcap-ng0 \
make libmagic-dev liblzma-dev liblz4-dev pkg-config autoconf libtool
```

---

## 🛠️ Установка Snort

### **Шаг 1: Создание рабочей директории**
```bash
sudo mkdir -p /opt/snort_src
cd /opt/snort_src
```

### **Шаг 2: Установка DAQ (Data Acquisition Library)**
```bash
# Скачивание DAQ
sudo wget https://www.snort.org/downloads/snort/daq-2.0.10.tar.gz
sudo tar -xvzf daq-2.0.10.tar.gz
cd daq-2.0.10

# Компиляция и установка
sudo ./configure
sudo make
sudo make install
```

### **Шаг 3: Установка Snort**
```bash
cd /opt/snort_src

# Скачивание Snort
sudo wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
sudo tar -xvzf snort-2.9.20.tar.gz
cd snort-2.9.20

# Компиляция и установка
sudo ./configure --enable-sourcefire
sudo make
sudo make install
```

### **Шаг 4: Настройка библиотек**
```bash
# Обновление путей библиотек
sudo ldconfig

# Создание символических ссылок
sudo ln -s /usr/local/bin/snort /usr/sbin/snort
```

---

## 📁 Настройка структуры каталогов

### **Создание необходимых директорий**
```bash
# Основные директории
sudo mkdir -p /etc/snort
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /etc/snort/preproc_rules
sudo mkdir -p /etc/snort/so_rules
sudo mkdir -p /var/log/snort

# Директории для правил
sudo mkdir -p /usr/local/lib/snort_dynamicrules
sudo mkdir -p /etc/snort/rules/iplists

# Создание необходимых файлов
sudo touch /etc/snort/rules/white_list.rules
sudo touch /etc/snort/rules/black_list.rules
sudo touch /etc/snort/rules/local.rules
```

### **Назначение прав**
```bash
sudo chmod -R 5775 /etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
sudo chown -R root:root /etc/snort
```

---

## ⚙️ Базовая конфигурация Snort

### **Шаг 1: Копирование конфигурационных файлов**
```bash
cd /opt/snort_src/snort-2.9.20

# Копирование конфигурационных файлов
sudo cp etc/*.conf* /etc/snort
sudo cp etc/*.map /etc/snort

# Копирование конфигурации по умолчанию
sudo cp snort.conf /etc/snort/snort.conf.default
```

### **Шаг 2: Настройка сетевого интерфейса**
```bash
# Определите ваш сетевой интерфейс
ip addr show

# Пример вывода:
# 2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
# INTERFACE="ens33"
```

### **Шаг 3: Редактирование основного конфигурационного файла**
```bash
sudo nano /etc/snort/snort.conf
```

**Внесите следующие изменения:**

```bash
# Установите правильные сетевые переменные (пример для домашней сети)
ipvar HOME_NET 192.168.1.0/24
ipvar EXTERNAL_NET !$HOME_NET

# Укажите DNS серверы
ipvar DNS_SERVERS 192.168.1.1

# Настройте пути к правилам
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
var WHITE_LIST_PATH /etc/snort/rules/iplists
var BLACK_LIST_PATH /etc/snort/rules/iplists

# Включите основные правила (раскомментируйте строки)
include $RULE_PATH/local.rules
```

### **Шаг 4: Создание тестовых правил**
```bash
sudo nano /etc/snort/rules/local.rules
```

**Добавьте тестовые правила:**
```bash
# ICMP правило для тестирования
alert icmp any any -> $HOME_NET any (msg:"ICMP test detected"; sid:1000001; rev:1;)

# SSH bruteforce detection
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 (msg:"SSH connection attempt"; flags:S; sid:1000002; rev:1;)

# Web attack detection
alert tcp $EXTERNAL_NET any -> $HOME_NET 80 (msg:"Web attack detected"; content:"/etc/passwd"; nocase; sid:1000003; rev:1;)
```

---

## 🔍 Проверка установки

### **Тест 1: Проверка версии Snort**
```bash
sudo snort -V
```

**Ожидаемый вывод:**
```
   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.20 GRE (Build 1)
   ''''    By Martin Roesch & The Snort Team
```

### **Тест 2: Проверка конфигурации**
```bash
sudo snort -T -c /etc/snort/snort.conf -i ens33
```

**Замените `ens33` на ваш интерфейс. Ожидаемый вывод:**
```
Snort successfully validated the configuration!
Snort exiting
```

---

## 🚀 Запуск Snort в различных режимах

### **Режим 1: Проверка пакетов (Packet Logger Mode)**
```bash
sudo snort -i ens33 -c /etc/snort/snort.conf -l /var/log/snort
```

### **Режим 2: IDS режим (Network Intrusion Detection Mode)**
```bash
sudo snort -i ens33 -c /etc/snort/snort.conf -A console -l /var/log/snort
```

### **Режим 3: Фоновый режим с выводом в syslog**
```bash
sudo snort -i ens33 -c /etc/snort/snort.conf -l /var/log/snort -D
```

---

## 🧪 Тестирование работы Snort

### **Тест 1: Генерация тестового трафика**
```bash
# Откройте два терминала

# Терминал 1: Запустите Snort в режиме реального времени
sudo snort -i ens33 -c /etc/snort/snort.conf -A console -l /var/log/snort

# Терминал 2: Отправьте тестовые пакеты
ping -c 3 192.168.1.1

# Или используйте nmap для сканирования
sudo nmap -sS 192.168.1.0/24
```

### **Тест 2: Проверка логов**
```bash
# Просмотр логов в реальном времени
sudo tail -f /var/log/snort/alert

# Просмотр собранных пакетов
sudo snort -r /var/log/snort/snort.log.1234567890
```

---

## 📊 Анализ результатов

### **Проверка созданных файлов**
```bash
# Проверьте логи
sudo ls -la /var/log/snort/

# Пример вывода:
# -rw------- 1 root root  1234 Dec 11 15:30 alert
# -rw------- 1 root root 56789 Dec 11 15:30 snort.log.1234567890
```

### **Анализ алертов**
```bash
sudo cat /var/log/snort/alert
```

**Пример вывода алерта:**
```
[**] [1:1000001:1] ICMP test detected [**]
[Classification: Misc activity] [Priority: 3] 
12/11-15:30:25.123456 192.168.1.100 -> 192.168.1.1
ICMP TTL:64 TOS:0x0 ID:12345 IpLen:20 DgmLen:84
```

---

## 🛡️ Дополнительная настройка для продакшн

### **Установка официальных правил Snort**
```bash
# Регистрация на snort.org и получение oinkcode
# Скачивание правил
cd /opt/snort_src
sudo wget https://www.snort.org/rules/snortrules-snapshot-29290.tar.gz?oinkcode=YOUR_OINKCODE -O snortrules.tar.gz
sudo tar -xvzf snortrules.tar.gz -C /etc/snort/
```

### **Настройка автоматического обновления правил**
```bash
# Создание скрипта обновления
sudo nano /opt/scripts/update_snort_rules.sh
```

```bash
#!/bin/bash
# Скрипт обновления правил Snort
OINKCODE="your_oinkcode_here"
RULES_URL="https://www.snort.org/rules/snortrules-snapshot-29290.tar.gz?oinkcode=$OINKCODE"

cd /tmp
wget -O snortrules.tar.gz "$RULES_URL"
tar -xvzf snortrules.tar.gz -C /etc/snort/
systemctl restart snort
```

```bash
sudo chmod +x /opt/scripts/update_snort_rules.sh

# Добавление в cron для ежедневного обновления
sudo crontab -e
# Добавьте строку: 0 2 * * * /opt/scripts/update_snort_rules.sh
```

### **Создание systemd службы**
```bash
sudo nano /etc/systemd/system/snort.service
```

```ini
[Unit]
Description=Snort NIDS Daemon
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -i ens33 -c /etc/snort/snort.conf -l /var/log/snort -D
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**Активация службы:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable snort
sudo systemctl start snort
sudo systemctl status snort
```

---

## 🔧 Утилиты для анализа логов Snort

### **Установка Barnyard2**
```bash
cd /opt/snort_src
sudo wget https://github.com/firnsy/barnyard2/archive/master.tar.gz -O barnyard2.tar.gz
sudo tar -xvzf barnyard2.tar.gz
cd barnyard2-master
sudo ./autogen.sh
sudo ./configure --with-mysql
sudo make
sudo make install
```

### **Использование PulledPork для управления правилами**
```bash
cd /opt/snort_src
sudo git clone https://github.com/shirkdog/pulledpork.git
cd pulledpork
sudo cp pulledpork.pl /usr/local/bin/
sudo cp etc/* /etc/snort/
sudo chmod +x /usr/local/bin/pulledpork.pl
```

---

## 🎯 Практическое задание: Проверка системы

### **Задание 1: Обнаружение сканирования портов**
```bash
# Добавьте правило в local.rules
echo 'alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"Port scan detected"; flags:S; detection_filter:track by_src, count 5, seconds 10; sid:1000004; rev:1;)' | sudo tee -a /etc/snort/rules/local.rules

# Перезапустите Snort
sudo systemctl restart snort

# Протестируйте сканированием
sudo nmap -sS 192.168.1.0/24
```

### **Задание 2: Обнаружение HTTP атак**
```bash
# Добавьте правило для обнаружения SQL инъекций
echo 'alert tcp $EXTERNAL_NET any -> $HOME_NET 80 (msg:"Possible SQL Injection"; content:"union"; nocase; content:"select"; nocase; sid:1000005; rev:1;)' | sudo tee -a /etc/snort/rules/local.rules
```

### **Задание 3: Создание отчета безопасности**
```bash
#!/bin/bash
# security_report.sh

echo "=== ОТЧЕТ БЕЗОПАСНОСТИ SNORT ==="
echo "Время: $(date)"
echo

# Статистика алертов
ALERT_COUNT=$(sudo grep -c "\[**\]" /var/log/snort/alert 2>/dev/null || echo "0")
echo "Всего алертов: $ALERT_COUNT"

# Топ правил
echo "Топ сработавших правил:"
sudo grep "\[**\]" /var/log/snort/alert 2>/dev/null | awk '{print $5}' | sort | uniq -c | sort -nr | head -5

# Статистика по IP
echo "Топ атакующих IP:"
sudo grep "\[**\]" /var/log/snort/alert 2>/dev/null | awk '{print $NF}' | sort | uniq -c | sort -nr | head -5
```

---

## ✅ Чеклист успешной установки

- [ ] Snort скомпилирован и установлен
- [ ] Конфигурационные файлы настроены
- [ ] Тестовые правила добавлены
- [ ] Snort запускается без ошибок
- [ ] Алерты генерируются при тестовом трафике
- [ ] Логи сохраняются корректно
- [ ] Systemd служба работает (опционально)
- [ ] Правила обновляются автоматически (опционально)

## 🆘 Устранение проблем

### **Проблема: "snort: error while loading shared libraries"**
```bash
# Решение: Обновить кэш библиотек
sudo ldconfig
```

### **Проблема: "Unable to open /etc/snort/snort.conf"**
```bash
# Решение: Проверить пути и права
sudo ls -la /etc/snort/
sudo chmod -R 644 /etc/snort/*.conf
```

### **Проблема: "No suitable interface found"**
```bash
# Решение: Проверить доступные интерфейсы
ip link show
# Использовать правильное имя интерфейса в конфигурации
```

Теперь ваша система Ubuntu 22.04 защищена системой обнаружения вторжений Snort! 🛡️