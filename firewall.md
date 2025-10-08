
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

