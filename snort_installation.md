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