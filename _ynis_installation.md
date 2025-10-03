# 🛡️ Комплексный аудит безопасности системы с помощью Lynis на Ubuntu 22.04

## 📋 Что такое Lynis?

**Lynis** — это мощный инструмент аудита безопасности с открытым исходным кодом, который проводит глубокую проверку системы Linux/macOS/UNIX. Он помогает выявить уязвимости и дает рекомендации по усилению безопасности.

---

## 🚀 Установка Lynis

### **Способ 1: Установка из официальных репозиториев**
```bash
# Обновление пакетов
sudo apt update && sudo apt upgrade -y

# Установка Lynis
sudo apt install -y lynis

# Проверка установки
lynis --version
```

### **Способ 2: Установка последней версии вручную**
```bash
# Скачивание последней версии
cd /tmp
wget https://downloads.cisofy.com/lynis/lynis-3.0.9.tar.gz

# Распаковка
tar xvf lynis-3.0.9.tar.gz
cd lynis

# Запуск без установки
./lynis audit system
```

---

## 🔍 Запуск комплексного аудита

### **Базовый аудит системы**
```bash
sudo lynis audit system
```

### **Расширенный аудит с подробным выводом**
```bash
sudo lynis audit system --verbose
```

### **Аудит с определенными категориями**
```bash
# Только проверки безопасности
sudo lynis audit system --tests-from-category "security"

# Проверка конкретных плагинов
sudo lynis audit system --plugin-list
```

---

## 📊 Анализ результатов аудита

### **Понимание вывода Lynis**

После завершения аудита вы увидите:

```
================================================================================

  Lynis 3.0.9

  Hardening and security scanning tool for Linux, macOS, and UNIX-based systems

  2007-2023, CISOfy - https://cisofy.com/lynis/
  Enterprise support available (compliance, plugins, interface, tools)

================================================================================

[+] Initializing program
...
```

### **Ключевые секции отчета:**

#### **1. Сводка тестов**
```
Tests: 291
Components: [...................]
```

#### **2. Результаты проверок**
```
[ OK ]      Firewall is active
[ WARNING ] No password set for single mode
[ SUGGESTION ] Set a password for single user mode [test: BOOT-5122] [details: ] [solution: run 'passwd' to set a password]
```

#### **3. Оценка безопасности**
```
Hardening index : 65 [#################----]
```

#### **4. Рекомендации**
```
  - Suggestions (25)
  - Warnings (3)
```

---

## 📋 Детальный разбор проверок Lynis

### **Основные категории проверок:**

| **Категория** | **Что проверяет** | **Важность** |
|---------------|-------------------|--------------|
| **Boot and services** | Загрузчики, сервисы | 🔴 Критическая |
| **Kernel** | Параметры ядра | 🔴 Критическая |
| **Memory and processes** | Память, процессы | 🟠 Высокая |
| **Users, groups and authentication** | Пользователи, аутентификация | 🔴 Критическая |
| **Shells** | Настройки оболочек | 🟡 Средняя |
| **File systems** | Файловые системы | 🟠 Высокая |
| **Storage** | Хранилище | 🟡 Средняя |
| **Networking** | Сетевая конфигурация | 🔴 Критическая |
| **Ports and packages** | Порти, пакеты | 🟠 Высокая |
| **Software** | Установленное ПО | 🟠 Высокая |
| **File permissions** | Права доступа к файлам | 🟠 Высокая |
| **Logging and files** | Логирование | 🟡 Средняя |
| **Scheduling** | Планировщики задач | 🟡 Средняя |
| **Cryptography** | Криптография | 🟠 Высокая |
| **Malware** | Антивирусная защита | 🟠 Высокая |

---

## 🛠️ Практическое руководство по аудиту

### **Шаг 1: Подготовка к аудиту**
```bash
# Создайте директорию для отчетов
sudo mkdir -p /var/log/lynis-reports
cd /var/log/lynis-reports

# Проверьте доступное место на диске
df -h
```

### **Шаг 2: Запуск полного аудита**
```bash
# Запуск с сохранением отчета
sudo lynis audit system --auditor "Ваше_Имя" --logfile /var/log/lynis-reports/audit-$(date +%Y%m%d).log
```

### **Шаг 3: Мониторинг прогресса**
Во время выполнения Lynis покажет прогресс:
```
[##-------] 10% (30/291) | Performing test: KRNL-5789
```

---

## 📈 Анализ и интерпретация результатов

### **Понимание оценок:**

- **🟢 0-49** - Низкий уровень безопасности
- **🟡 50-74** - Средний уровень безопасности  
- **🟠 75-89** - Высокий уровень безопасности
- **🔴 90-100** - Очень высокий уровень безопасности

### **Типы рекомендаций:**

```bash
# Просмотр всех рекомендаций
sudo grep -E "(Suggestion|Warning)" /var/log/lynis-reports/audit-*.log

# Критические рекомендации
sudo grep "CRITICAL" /var/log/lynis-reports/audit-*.log

# Предложения по улучшению
sudo grep "suggestion" /var/log/lynis-reports/audit-*.log -i
```

---

## 🔧 Автоматизация аудитов

### **Создание скрипта для регулярного аудита**
```bash
sudo nano /opt/scripts/lynis-audit.sh
```

```bash
#!/bin/bash
# Скрипт автоматического аудита Lynis

DATE=$(date +%Y%m%d)
LOG_DIR="/var/log/lynis-reports"
LOG_FILE="$LOG_DIR/lynis-audit-$DATE.log"
REPORT_FILE="$LOG_DIR/lynis-report-$DATE.txt"

# Создание директории для логов
mkdir -p $LOG_DIR

echo "=== ЗАПУСК АУДИТА LYNIS ===" > $REPORT_FILE
echo "Дата: $(date)" >> $REPORT_FILE
echo "Система: $(hostname)" >> $REPORT_FILE
echo >> $REPORT_FILE

# Запуск аудита
echo "Запуск комплексного аудита..."
sudo lynis audit system --logfile $LOG_FILE --report-file $REPORT_FILE

# Добавление сводки в отчет
echo "=== СВОДКА РЕКОМЕНДАЦИЙ ===" >> $REPORT_FILE
sudo grep -c "Suggestion" $LOG_FILE >> $REPORT_FILE
sudo grep -c "Warning" $LOG_FILE >> $REPORT_FILE

echo "Аудит завершен. Отчет: $REPORT_FILE"
```

```bash
# Сделайте скрипт исполняемым
sudo chmod +x /opt/scripts/lynis-audit.sh

# Запустите скрипт
sudo /opt/scripts/lynis-audit.sh
```

### **Настройка регулярных аудитов через cron**
```bash
sudo crontab -e

# Добавьте строку для еженедельного аудита
0 2 * * 0 /opt/scripts/lynis-audit.sh > /dev/null 2>&1

# Или ежедневный аудит в 3:00
0 3 * * * /opt/scripts/lynis-audit.sh > /dev/null 2>&1
```

---

## 🎯 Практические примеры исправления проблем

### **Пример 1: Исправление предупреждения о пароле single user mode**
```bash
# Установка пароля для single user mode
sudo passwd root
```

### **Пример 2: Настройка фаервола**
```bash
# Установка и настройка UFW
sudo apt install ufw
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
```

### **Пример 3: Усиление безопасности SSH**
```bash
sudo nano /etc/ssh/sshd_config

# Добавьте/измените строки:
PermitRootLogin no
PasswordAuthentication no
Protocol 2
MaxAuthTries 3
```

### **Пример 4: Настройка автоматических обновлений**
```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

---

## 📊 Создание сводного отчета

### **Скрипт для генерации красивого отчета**
```bash
sudo nano /opt/scripts/lynis-report-generator.sh
```

```bash
#!/bin/bash
# Генератор отчетов Lynis

LOG_FILE="$1"
REPORT_FILE="/tmp/lynis-summary-$(date +%Y%m%d).html"

if [ -z "$LOG_FILE" ]; then
    echo "Использование: $0 <lynis-log-file>"
    exit 1
fi

{
echo "<html>"
echo "<head>"
echo "<title>Отчет безопасности Lynis</title>"
echo "<style>"
echo "body { font-family: Arial, sans-serif; margin: 20px; }"
echo ".critical { color: red; font-weight: bold; }"
echo ".warning { color: orange; }"
echo ".suggestion { color: blue; }"
echo ".ok { color: green; }"
echo "</style>"
echo "</head>"
echo "<body>"
echo "<h1>Отчет безопасности Lynis</h1>"
echo "<p>Дата генерации: $(date)</p>"
echo "<p>Файл лога: $LOG_FILE</p>"
echo "<hr>"

# Критические предупреждения
echo "<h2>🔴 Критические проблемы</h2>"
grep -i "critical" "$LOG_FILE" | while read line; do
    echo "<p class='critical'>$line</p>"
done

# Предупреждения
echo "<h2>🟠 Предупреждения</h2>"
grep "Warning" "$LOG_FILE" | while read line; do
    echo "<p class='warning'>$line</p>"
done

# Предложения
echo "<h2>🔵 Рекомендации</h2>"
grep "Suggestion" "$LOG_FILE" | head -20 | while read line; do
    echo "<p class='suggestion'>$line</p>"
done

echo "</body>"
echo "</html>"
} > "$REPORT_FILE"

echo "HTML отчет создан: $REPORT_FILE"
```

```bash
sudo chmod +x /opt/scripts/lynis-report-generator.sh

# Использование
sudo /opt/scripts/lynis-report-generator.sh /var/log/lynis-reports/audit-20231211.log
```

---

## 🔍 Расширенные возможности Lynis

### **Профилирование системы**
```bash
# Создание профиля системы
sudo lynis show profiles

# Аудит с определенным профилем
sudo lynis audit system --profile "server"
```

### **Проверка на соответствие стандартам**
```bash
# Проверка соответствия CIS benchmarks
sudo lynis audit system --tests-from-group "cis"

# Проверка конкретного стандарта
sudo lynis audit system --tests-from-group "cis_level_1"
```

### **Интеграция с мониторингом**
```bash
# Экспорт результатов в JSON
sudo lynis audit system --report-file /tmp/lynis-report.json --format json

# Экспорт в CSV
sudo lynis audit system --report-file /tmp/lynis-report.csv --format csv
```

---

## 📝 Чеклист действий после аудита

### **Немедленные действия:**
- [ ] Установить пароль для single user mode
- [ ] Настроить и включить фаервол
- [ ] Отключить root вход по SSH
- [ ] Настроить автоматические обновления
- [ ] Проверить права доступа к критическим файлам

### **Среднесрочные улучшения:**
- [ ] Настроить централизованное логирование
- [ ] Внедрить двухфакторную аутентификацию
- [ ] Настроить мониторинг целостности файлов
- [ ] Регулярно обновлять правила Lynis

### **Долгосрочная стратегия:**
- [ ] Автоматизировать регулярные аудиты
- [ ] Интегрировать с SIEM системой
- [ ] Создать процессы реагирования на инциденты
- [ ] Регулярно пересматривать политики безопасности

---

## 🎯 Пример полного рабочего процесса

### **День 1: Первоначальный аудит**
```bash
sudo lynis audit system --logfile /var/log/lynis-initial.log
```

### **День 2: Анализ и приоритизация**
```bash
# Анализ результатов
grep -c "Warning" /var/log/lynis-initial.log
grep -c "Suggestion" /var/log/lynis-initial.log

# Создание плана действий
sudo grep "Warning\|Suggestion" /var/log/lynis-initial.log > /tmp/action-plan.txt
```

### **День 3-7: Реализация улучшений**
```bash
# Постепенное внедрение исправлений
# Мониторинг прогресса
```

### **День 8: Контрольный аудит**
```bash
sudo lynis audit system --logfile /var/log/lynis-followup.log
```

### **Сравнение результатов:**
```bash
# Сравнение оценок безопасности
echo "Начальная оценка:"
grep "Hardening index" /var/log/lynis-initial.log

echo "Текущая оценка:"
grep "Hardening index" /var/log/lynis-followup.log
```

---

## ✅ Заключение

Lynis — это мощный инструмент для комплексного аудита безопасности системы. Регулярное использование позволяет:

- 🎯 Выявлять уязвимости безопасности
- 📊 Оценивать текущий уровень защиты  
- 🔧 Получать конкретные рекомендации по улучшению
- 📈 Отслеживать прогресс в усилении безопасности
- 🚀 Автоматизировать процессы аудита

**Рекомендуемая частота аудитов:**
- 🔴 Продакшн-серверы: еженедельно
- 🟡 Тестовые среды: ежемесячно
- 🟢 Рабочие станции: ежеквартально

Начните с базового аудита сегодня и постепенно внедряйте рекомендации для создания безопасной и защищенной системы! 🛡️