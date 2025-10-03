# 🚀 Подробное руководство по запуску скриптов безопасности на Ubuntu 22.04

## 📋 Подготовка системы

### **1. Проверка прав доступа**
```bash
# Убедитесь, что у вас есть права sudo
sudo whoami

# Проверьте версию Ubuntu
lsb_release -a
```

### **2. Установка необходимых утилит**
```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка дополнительных утилит
sudo apt install -y net-tools grep awk sed curl wget
```

---

## 🛠️ Создание и настройка скриптов

### **Шаг 1: Создание директории для скриптов**
```bash
# Создайте директорию для скриптов безопасности
sudo mkdir -p /opt/security-scripts
cd /opt/security-scripts
```

### **Шаг 2: Создание Скрипта 1 - Комплексный анализ безопасности**

```bash
# Создайте файл скрипта
sudo nano /opt/security-scripts/security_analysis.sh
```

**Вставьте содержимое Скрипта 1, затем:**

```bash
# Сделайте скрипт исполняемым
sudo chmod +x /opt/security-scripts/security_analysis.sh

# Проверьте синтаксис скрипта
bash -n /opt/security-scripts/security_analysis.sh
```

### **Шаг 3: Создание Скрипта 2 - Мониторинг в реальном времени**

```bash
# Создайте файл скрипта
sudo nano /opt/security-scripts/realtime_monitor.sh
```

**Вставьте содержимое Скрипта 2, затем:**

```bash
# Сделайте скрипт исполняемым
sudo chmod +x /opt/security-scripts/realtime_monitor.sh

# Проверьте синтаксис
bash -n /opt/security-scripts/realtime_monitor.sh
```

### **Шаг 4: Создание Скрипта 3 - Ежедневный отчет**

```bash
# Создайте файл скрипта
sudo nano /opt/security-scripts/daily_report.sh
```

**Вставьте содержимое Скрипта 3, затем:**

```bash
sudo chmod +x /opt/security-scripts/daily_report.sh
bash -n /opt/security-scripts/daily_report.sh
```

### **Шаг 5: Создание системы оповещений**

```bash
# Создайте файл скрипта
sudo nano /opt/security-scripts/alert_system.sh
```

**Вставьте содержимое alert-системы, затем:**

```bash
sudo chmod +x /opt/security-scripts/alert_system.sh
bash -n /opt/security-scripts/alert_system.sh
```

---

## 🚀 Запуск скриптов

### **Запуск Скрипта 1 см. Раздел 🔧 Практические скрипты для анализа: Комплексный анализ безопасности**
```bash
# Запуск с правами суперпользователя (рекомендуется)
sudo /opt/security-scripts/security_analysis.sh

# Или перейдите в директорию и запустите
cd /opt/security-scripts
sudo ./security_analysis.sh
```

**Ожидаемый вывод:**
```
=== КОМПЛЕКСНЫЙ АНАЛИЗ БЕЗОПАСНОСТИ ===
Время: Пн дек 11 14:30:25 UTC 2023

🔐 АНАЛИЗ АУТЕНТИФИКАЦИИ:
----------------------------
Неудачные попытки входа: 15
Успешные входы: 3
Топ IP для блокировки:
   5 192.168.1.50
   3 103.216.45.12
...
```

### **Запуск Скрипта 2 см. Раздел 🔧 Практические скрипты для анализа: Мониторинг в реальном времени**
```bash
# Запуск в фоновом режиме
sudo nohup /opt/security-scripts/realtime_monitor.sh &

# Запуск в текущей сессии
sudo /opt/security-scripts/realtime_monitor.sh

# Для остановки нажмите Ctrl+C
```

**Ожидаемое поведение:** Скрипт будет показывать события безопасности в реальном времени.

### **Запуск Скрипта 3 см. Раздел 🔧 Практические скрипты для анализа : Ежедневный отчет**
```bash
# Ручной запуск
sudo /opt/security-scripts/daily_report.sh

# Проверьте созданный отчет
cat /tmp/security_report_20231211.txt
```

---

## ⚡ Автоматизация запуска

### **Настройка cron для ежедневных отчетов**
```bash
# Откройте crontab для редактирования
sudo crontab -e

# Добавьте строку для ежедневного запуска в 6:00 утра
0 6 * * * /opt/security-scripts/daily_report.sh

# Или для запуска каждый час
0 * * * * /opt/security-scripts/security_analysis.sh >> /var/log/security_analysis.log
```

### **Создание systemd службы для мониторинга**
```bash
# Создайте service файл
sudo nano /etc/systemd/system/security-monitor.service
```

**Содержимое файла:**
```ini
[Unit]
Description=Security Monitor Service
After=network.target

[Service]
Type=simple
ExecStart=/opt/security-scripts/realtime_monitor.sh
Restart=always
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
```

**Активация службы:**
```bash
# Перезагрузите systemd
sudo systemctl daemon-reload

# Включите автозапуск
sudo systemctl enable security-monitor.service

# Запустите службу
sudo systemctl start security-monitor.service

# Проверьте статус
sudo systemctl status security-monitor.service
```

---

## 🔧 Решение возможных проблем

### **Проблема 1: "Permission denied"**
```bash
# Решение: Проверьте права доступа
sudo chmod +x /opt/security-scripts/*.sh
sudo chown root:root /opt/security-scripts/*.sh
```

### **Проблема 2: "No such file or directory" для логов**
```bash
# Решение: Проверьте существование лог-файлов
sudo ls -la /var/log/auth.log
sudo ls -la /var/log/nginx/access.log

# Если nginx не установлен, установите его
sudo apt install -y nginx
```

### **Проблема 3: Скрипт не выполняется**
```bash
# Проверьте шебанг (первая строка)
head -1 /opt/security-scripts/security_analysis.sh
# Должно быть: #!/bin/bash

# Проверьте права
ls -la /opt/security-scripts/security_analysis.sh
# Должно быть: -rwxr-xr-x
```

### **Проблема 4: Нет доступа к сетевой информации**
```bash
# Убедитесь, что net-tools установлен
sudo apt install -y net-tools

# Проверьте доступность команд
which netstat
which ss
```

---

## 📊 Тестирование скриптов

### **Тест 1: Проверка работы анализа безопасности**
```bash
# Запустите скрипт и проверьте вывод
sudo /opt/security-scripts/security_analysis.sh | tee /tmp/test_output.log

# Проверьте, что все секции работают
grep -E "(АНАЛИЗ|Успешные|Неудачные|Подозрительные)" /tmp/test_output.log
```

### **Тест 2: Создание тестовых данных для мониторинга**
```bash
# Создайте тестовое событие в auth.log
echo "$(date) test sshd[12345]: Failed password for root from 192.168.1.100 port 22" | sudo tee -a /var/log/auth.log

# Запустите мониторинг и убедитесь, что событие обнаруживается
sudo /opt/security-scripts/realtime_monitor.sh
```

### **Тест 3: Проверка ежедневного отчета**
```bash
# Запустите отчет
sudo /opt/security-scripts/daily_report.sh

# Проверьте созданный файл
ls -la /tmp/security_report_*.txt

# Просмотрите содержимое
cat /tmp/security_report_$(date +%Y%m%d).txt
```

---

## 🎯 Дополнительные настройки для улучшения работы

### **Настройка ротации логов безопасности**
```bash
# Создайте конфигурацию logrotate для security alerts
sudo nano /etc/logrotate.d/security-alerts
```

**Содержимое:**
```
/var/log/security_alerts.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 644 root root
}
```

### **Создание алиасов для быстрого доступа**
```bash
# Добавьте в ~/.bashrc или /etc/bash.bashrc
alias security-scan='sudo /opt/security-scripts/security_analysis.sh'
alias security-monitor='sudo /opt/security-scripts/realtime_monitor.sh'
alias security-report='sudo /opt/security-scripts/daily_report.sh'

# Примените изменения
source ~/.bashrc
```

### **Настройка отправки email оповещений**
```bash
# Установите почтовую утилиту
sudo apt install -y mailutils

# Раскомментируйте строки в alert_system.sh
# echo "$message" | mail -s "Security Alert: $level" admin@company.com
```

---

## 📈 Мониторинг производительности

### **Скрипт для проверки нагрузки от мониторинга**
```bash
#!/bin/bash
# monitoring_performance.sh

echo "=== МОНИТОРИНГ НАГРУЗКИ СКРИПТОВ ==="

# Потребление памяти
echo "Память:"
ps aux | grep -E "(realtime_monitor|alert_system)" | grep -v grep | awk '{print $4 "% memory"}'

# Потребление CPU
echo "CPU:"
ps aux | grep -E "(realtime_monitor|alert_system)" | grep -v grep | awk '{print $3 "% CPU"}'

# Размер логов
echo "Размер логов безопасности:"
du -sh /var/log/security_alerts.log 2>/dev/null || echo "Файл логов не существует"
```

```bash
# Создайте и запустите
sudo nano /opt/security-scripts/monitoring_performance.sh
sudo chmod +x /opt/security-scripts/monitoring_performance.sh
sudo /opt/security-scripts/monitoring_performance.sh
```

---

## ✅ Чеклист успешной установки

- [ ] Все скрипты созданы в `/opt/security-scripts/`
- [ ] Права доступа установлены (chmod +x)
- [ ] Скрипт 1 запускается и показывает анализ
- [ ] Скрипт 2 работает в реальном времени
- [ ] Скрипт 3 создает ежедневные отчеты
- [ ] Systemd служба работает (если настроена)
- [ ] Cron задания добавлены для автоматизации
- [ ] Логи ротируются корректно

## 🆘 Команды для диагностики проблем

```bash
# Проверить все скрипты на синтаксические ошибки
for script in /opt/security-scripts/*.sh; do
    echo "Проверка $script:"
    bash -n "$script" && echo "✅ OK" || echo "❌ Ошибка"
done

# Проверить логи скриптов
sudo tail -f /var/log/syslog | grep security-scripts

# Проверить использование ресурсов
top -p $(pgrep -f security-scripts)
```

# 🔍 Анализ логов для выявления угроз: Полное руководство

## 📊 Таблица кодов опасностей и приоритетов

### **Система классификации угроз:**

| **Код** | **Уровень опасности** | **Цвет** | **Описание** | **Примеры в логах** |
|---------|----------------------|----------|--------------|-------------------|
| **CRIT-001** | Критический | 🔴 | Прямая компрометация системы | Успешный взлом, бэкдоры |
| **CRIT-002** | Критический | 🔴 | Привилегированный доступ | root-команды, sudo |
| **HIGH-001** | Высокий | 🟠 | Попытки взлома | Bruteforce, SQL-инъекции |
| **HIGH-002** | Высокий | 🟠 | Подозрительные процессы | Неизвестные исполняемые файлы |
| **MED-001** | Средний | 🟡 | Сканирование/разведка | Port scanning, dir traversal |
| **MED-002** | Средний | 🟡 | Аномальная активность | Необычное время работы |
| **LOW-001** | Низкий | 🔵 | Подозрительные шаблоны | Множественные ошибки |
| **INFO-001** | Информационный | ⚪ | Нормальная активность | Легитимный доступ |

---

## 📋 Методика анализа логов

### **1. Анализ аутентификации (Auth Logs)**

#### **Критические индикаторы:**
```bash
# Просмотр логов аутентификации
sudo tail -f /var/log/auth.log

# Поиск успешных взломов
grep -i "accepted" /var/log/auth.log
grep -i "session opened" /var/log/auth.log

# Поиск подозрительных активностей
grep -i "failed" /var/log/auth.log | head -20
grep -i "invalid" /var/log/auth.log
```

#### **Шаблоны для обнаружения:**
```bash
# Bruteforce атаки (HIGH-001)
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Успешный взлом (CRIT-001)
grep -E "(session opened|accepted)" /var/log/auth.log | grep -v "sudo\|cron"

# Подозрительные пользователи (CRIT-002)
grep "new user\|useradd" /var/log/auth.log
```

### **2. Анализ веб-сервера (Web Server Logs)**

#### **Nginx/Apache логи:**
```bash
# Поиск SQL-инъекций (HIGH-001)
grep -E "(union.*select|select.*from|1=1|or 1=1)" /var/log/nginx/access.log

# Поиск XSS атак (HIGH-001)
grep -E "(<script>|alert\(|onload=|onerror=)" /var/log/nginx/access.log

# Поиск path traversal (MED-001)
grep -E "(\.\./|\.\.\\|etc/passwd|bin/sh)" /var/log/nginx/access.log

# Сканирование директорий (MED-001)
grep -E "(admin|phpmyadmin|wp-admin|\.git|\.env)" /var/log/nginx/access.log
```

#### **Анализ частоты запросов:**
```bash
# Топ IP адресов по количеству запросов
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -20

# Обнаружение DDoS (HIGH-002)
awk '{print $4}' /var/log/nginx/access.log | cut -d: -f1 | uniq -c | sort -nr
```

### **3. Анализ системных логов (System Logs)**

#### **Syslog анализ:**
```bash
# Подозрительные команды (CRIT-002)
grep -E "(wget|curl|bash -i|/bin/sh|/bin/bash)" /var/log/syslog

# Необычные процессы (HIGH-002)
grep -E "(tmp/|/var/tmp|/dev/shm|\.hidden)" /var/log/syslog

# Изменение прав доступа (MED-002)
grep -E "(chmod|chown|chattr)" /var/log/syslog | grep -v "CRON"
```

---

## 🛡️ Таблица сигнатур угроз

### **Критические сигнатуры (CRIT):**

| **Сигнатура** | **Код** | **Описание** | **Команда для поиска** |
|---------------|---------|--------------|----------------------|
| **Успешный SSH взлом** | CRIT-001 | Неавторизованный успешный вход | `grep "Accepted" /var/log/auth.log \| grep -v "192.168.1."` |
| **Выполнение от root** | CRIT-002 | Подозрительные root команды | `grep "COMMAND" /var/log/auth.log \| grep "root"` |
| **Установка бэкдора** | CRIT-001 | Загрузка и выполнение скриптов | `grep -E "(wget.*sh\|curl.*bash)" /var/log/syslog` |

### **Высокорисковые сигнатуры (HIGH):**

| **Сигнатура** | **Код** | **Описание** | **Команда для поиска** |
|---------------|---------|--------------|----------------------|
| **Bruteforce SSH** | HIGH-001 | Множественные неудачные попытки | `grep "Failed password" /var/log/auth.log \| awk '{print $11}' \| sort \| uniq -c \| sort -nr` |
| **SQL инъекции** | HIGH-001 | Попытки SQL-атак | `grep -E "(union.*select\|drop table)" /var/log/nginx/access.log` |
| **Подозрительные процессы** | HIGH-002 | Исполнение из temp директорий | `ps aux \| grep -E "(tmp/\|/var/tmp)"` |

### **Средние сигнатуры (MED):**

| **Сигнатура** | **Код** | **Описание** | **Команда для поиска** |
|---------------|---------|--------------|----------------------|
| **Сканирование портов** | MED-001 | Множественные подключения | `netstat -tun \| awk '{print $5}' \| cut -d: -f1 \| sort \| uniq -c \| sort -nr` |
| **Реконносцировка** | MED-001 | Поиск уязвимых директорий | `grep -E "(phpmyadmin\|wp-admin\|\.git)" /var/log/nginx/access.log` |
| **Аномальное время** | MED-002 | Активность в нерабочее время | `grep "22:[0-5][0-9]:\|03:[0-5][0-9]:" /var/log/auth.log` |

---

## 🔧 Практические скрипты для анализа

### **Скрипт 1: Комплексный анализ безопасности**
```bash
#!/bin/bash
echo "=== КОМПЛЕКСНЫЙ АНАЛИЗ БЕЗОПАСНОСТИ ==="
echo "Время: $(date)"
echo

# 1. Анализ аутентификации
echo "🔐 АНАЛИЗ АУТЕНТИФИКАЦИИ:"
echo "----------------------------"
failed_logins=$(grep "Failed password" /var/log/auth.log 2>/dev/null | wc -l)
success_logins=$(grep "Accepted" /var/log/auth.log 2>/dev/null | wc -l)

echo "Неудачные попытки входа: $failed_logins"
echo "Успешные входы: $success_logins"

# Bruteforce detection
echo "Топ IP для блокировки:"
grep "Failed password" /var/log/auth.log 2>/dev/null | awk '{print $11}' | sort | uniq -c | sort -nr | head -5

echo

# 2. Анализ веб-трафика
echo "🌐 АНАЛИЗ ВЕБ-ТРАФИКА:"
echo "-----------------------"
if [ -f "/var/log/nginx/access.log" ]; then
    suspicious_requests=$(grep -E "(union|select|1=1|script|etc/passwd)" /var/log/nginx/access.log | wc -l)
    echo "Подозрительные HTTP запросы: $suspicious_requests"
    
    echo "Топ атакующих IP:"
    grep -E "(union|select|1=1)" /var/log/nginx/access.log 2>/dev/null | awk '{print $1}' | sort | uniq -c | sort -nr | head -5
fi

echo

# 3. Анализ процессов
echo "⚙️ АНАЛИЗ ПРОЦЕССОВ:"
echo "-------------------"
suspicious_procs=$(ps aux | grep -E "(tmp/|/var/tmp|/dev/shm|\.hidden)" | grep -v grep | wc -l)
echo "Подозрительные процессы: $suspicious_procs"

# 4. Сетевой анализ
echo "🌐 СЕТЕВОЙ АНАЛИЗ:"
echo "------------------"
echo "Активные подключения:"
netstat -tun | grep ESTABLISHED | wc -l

echo "Необычные порты:"
netstat -tuln | awk '{print $4}' | grep -E ":(4444|1337|31337)" | wc -l
```

### **Скрипт 2: Мониторинг в реальном времени**
```bash
#!/bin/bash
# Мониторинг угроз в реальном времени
echo "🚨 МОНИТОРИНГ УГРОЗ В РЕАЛЬНОМ ВРЕМЕНИ"
echo "Нажмите Ctrl+C для остановки"

tail -f /var/log/auth.log /var/log/nginx/access.log /var/log/syslog 2>/dev/null | \
while read line; do
    # Критические угрозы
    if echo "$line" | grep -q -E "(Accepted|session opened)"; then
        echo "🔴 CRIT-001: Успешный вход: $line"
    fi
    
    # Высокорисковые угрозы
    if echo "$line" | grep -q -E "(union.*select|1=1|script)"; then
        echo "🟠 HIGH-001: Веб-атака: $line"
    fi
    
    # Bruteforce попытки
    if echo "$line" | grep -q "Failed password"; then
        echo "🟡 HIGH-002: Неудачный вход: $line"
    fi
done
```

### **Скрипт 3: Ежедневный отчет**
```bash
#!/bin/bash
# Генерация ежедневного отчета безопасности
REPORT_FILE="/tmp/security_report_$(date +%Y%m%d).txt"

{
echo "=== ЕЖЕДНЕВНЫЙ ОТЧЕТ БЕЗОПАСНОСТИ ==="
echo "Дата: $(date)"
echo "Сервер: $(hostname)"
echo

echo "📊 СВОДКА УГРОЗ:"
echo "================"

# Сбор статистики
FAILED_SSH=$(grep "Failed password" /var/log/auth.log | wc -l)
SUCCESS_SSH=$(grep "Accepted" /var/log/auth.log | wc -l)
SQL_INJECTIONS=$(grep -E "(union.*select|1=1)" /var/log/nginx/access.log 2>/dev/null | wc -l)
SCAN_ATTEMPTS=$(grep -E "(phpmyadmin|wp-admin|\.git)" /var/log/nginx/access.log 2>/dev/null | wc -l)

echo "🔐 SSH Аутентификация:"
echo "  - Неудачные попытки: $FAILED_SSH"
echo "  - Успешные входы: $SUCCESS_SSH"
echo

echo "🌐 Веб-атаки:"
echo "  - SQL инъекции: $SQL_INJECTIONS"
echo "  - Сканирование: $SCAN_ATTEMPTS"
echo

echo "🚨 ТОП УГРОЗ:"
echo "============="
echo "1. Bruteforce IP:"
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head -3

echo
echo "2. Веб-атакующие:"
grep -E "(union|select|1=1)" /var/log/nginx/access.log 2>/dev/null | awk '{print $1}' | sort | uniq -c | sort -nr | head -3

} > $REPORT_FILE

echo "Отчет сохранен: $REPORT_FILE"
```

---

## 📈 Визуализация и мониторинг

### **Настройка alert-системы:**
```bash
#!/bin/bash
# Система оповещений о критических событиях

# Функция отправки alert
send_alert() {
    local level=$1
    local message=$2
    echo "[$(date)] $level: $message" >> /var/log/security_alerts.log
    
    # Отправка email (раскомментировать при настройке почты)
    # echo "$message" | mail -s "Security Alert: $level" admin@company.com
}

# Мониторинг в реальном времени
tail -f /var/log/auth.log | while read line; do
    # CRITICAL alerts
    if echo "$line" | grep -q "Accepted.*root"; then
        send_alert "CRIT-001" "Root login detected: $line"
    fi
    
    # HIGH alerts
    if echo "$line" | grep -q "Failed password.*root"; then
        send_alert "HIGH-001" "Bruteforce attempt on root: $line"
    fi
done
```

---

## 🎯 Шаблоны ответов на инциденты

### **При обнаружении угрозы:**

| **Код угрозы** | **Немедленные действия** | **Долгосрочные меры** |
|----------------|-------------------------|----------------------|
| **CRIT-001** | Блокировка IP, Отключение пользователя, Аудит системы | 2FA, Усиление аутентификации |
| **HIGH-001** | Rate limiting, Блокировка IP, Мониторинг | WAF, Обновление правил |
| **MED-001** | Логирование, Уведомление | ACL настройка, Сегментация сети |

---

## 💡 Рекомендации по внедрению

1. **Настройте централизованное логирование**
2. **Внедрите автоматические скрипты мониторинга**
3. **Настройте систему оповещений**
4. **Регулярно проводите анализ исторических данных**
5. **Обновляйте сигнатуры угроз**

Эта система позволит эффективно выявлять и классифицировать угрозы безопасности на ваших серверах.

