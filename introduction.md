# 🚀 Практическое руководство по работе с утилитой `dig`

## 📋 Введение в `dig`

**Dig** (Domain Information Groper) - мощная утилита для диагностики DNS. В отличие от `nslookup`, она предоставляет более подробную информацию и гибкость в использовании.

---

## 🛠️ Установка и базовая проверка

### **Установка dig:**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install dnsutils

# CentOS/RHEL
sudo yum install bind-utils

# Проверка установки
dig -v
```

### **Базовая структура команды:**
```bash
dig [@сервер] [домен] [тип_записи] [опции]
```

---

## 🔍 БАЗОВЫЕ КОМАНДЫ

### **1. Простой запрос A-записи**
```bash
dig example.com

# Сокращенный вывод
dig example.com +short
```
**Что узнаем:** IP-адрес домена

### **2. Запрос конкретного типа записи**
```bash
# A запись (IPv4)
dig example.com A

# AAAA запись (IPv6)
dig example.com AAAA

# MX запись (почтовые серверы)
dig example.com MX

# NS запись (серверы имен)
dig example.com NS

# TXT запись (текстовые записи)
dig example.com TXT

# CNAME запись (алиасы)
dig www.example.com CNAME
```

### **3. Запрос к конкретному DNS серверу**
```bash
# Использование Google DNS
dig @8.8.8.8 example.com

# Использование Cloudflare DNS
dig @1.1.1.1 example.com

# Запрос к конкретному authoritative серверу
dig @ns1.google.com google.com
```

---

## 🎯 ПРАКТИЧЕСКОЕ ЗАДАНИЕ 1: Базовый анализ домена

### **Задача:** Проанализировать домен `google.com`

```bash
# 1. Основной IP адрес
dig google.com A +short

# 2. IPv6 адрес
dig google.com AAAA +short

# 3. Почтовые серверы
dig google.com MX

# 4. Серверы имен
dig google.com NS

# 5. Все записи сразу
dig google.com ANY
```

### **Ожидаемый результат:**
```bash
# Пример вывода
172.217.16.206
smtp.google.com.
ns1.google.com.
ns2.google.com.
```

---

## 🔎 РАСШИРЕННЫЕ ВОЗМОЖНОСТИ

### **4. Трассировка DNS запроса**
```bash
# Полная трассировка
dig google.com +trace

# Сокращенная трассировка
dig google.com +trace +short
```
**Что покажет:** Весь путь запроса от корневых серверов до конечного

### **5. Проверка DNSSEC**
```bash
# Проверка DNSSEC подписи
dig google.com DNSKEY

# Проверка целостности
dig google.com DS
```

### **6. Рекурсивный запрос vs Authoritative**
```bash
# Рекурсивный запрос (через локальный DNS)
dig example.com

# Прямой запрос к authoritative серверу
dig @ns1.example.com example.com
```

---

## 🛡️ ПРАКТИЧЕСКОЕ ЗАДАНИЕ 2: Анализ безопасности DNS

### **Задача:** Проверить безопасность домена

```bash
# 1. Проверить DNSSEC
dig example.com DNSKEY +dnssec

# 2. Проверить SPF запись (защита от спама)
dig example.com TXT | grep "v=spf1"

# 3. Проверить DMARC политику
dig _dmarc.example.com TXT

# 4. Проверить DKIM запись
dig selector._domainkey.example.com TXT

# 5. Проверить возможность зонного трансфера
dig example.com AXFR
```

### **Интерпретация результатов:**
- **DNSSEC**: Должны быть подписанные записи
- **SPF**: Должна быть настроена политика отправки почты
- **AXFR**: Должен возвращать отказ (защита от утечки зоны)

---

## 📊 ФОРМАТИРОВАНИЕ ВЫВОДА

### **Полезные опции форматирования:**
```bash
# Только ответ (идеально для скриптов)
dig example.com +noall +answer

# Короткий вывод
dig example.com +short

# Без комментариев
dig example.com +nocomments

# Статистика времени
dig example.com +stats

# Только секция ответа
dig example.com +noquestion +noauthority +noadditional
```

### **Пример красивого вывода:**
```bash
dig example.com +noall +answer
```
**Вывод:**
```
example.com.        86400   IN  A   93.184.216.34
example.com.        86400   IN  AAAA    2606:2800:220:1:248:1893:25c8:1946
```

---

## 🕵️ ПРАКТИЧЕСКОЕ ЗАДАНИЕ 3: Диагностика проблем DNS

### **Симптом:** Сайт не открывается, подозрение на проблемы с DNS

```bash
# 1. Проверить базовое разрешение
dig problem-site.com

# 2. Сравнить с публичным DNS
dig @8.8.8.8 problem-site.com
dig @1.1.1.1 problem-site.com

# 3. Проверить TTL записей
dig problem-site.com +ttlid

# 4. Проверить наличие CNAME
dig problem-site.com CNAME

# 5. Трассировка для диагностики
dig problem-site.com +trace
```

### **Анализ результатов:**
- Сравнить ответы разных DNS серверов
- Проверить TTL (возможно кеширование старой записи)
- Найти где обрывается трассировка

---

## 🌐 РАБОТА С РАЗНЫМИ ТИПАМИ ЗАПИСЕЙ

### **7. Специфические типы записей:**
```bash
# SOA запись (информация о зоне)
dig example.com SOA

# SRV запись (сервисы)
dig _sip._tcp.example.com SRV

# PTR запись (обратный DNS)
dig -x 8.8.8.8

# CAA запись (SSL сертификаты)
dig example.com CAA

# TLSA запись (DANE)
dig _443._tcp.example.com TLSA
```

---

## ⚡ ПРАКТИЧЕСКОЕ ЗАДАНИЕ 4: Мониторинг DNS изменений

### **Задача:** Написать скрипт для мониторинга изменений DNS

```bash
#!/bin/bash
DOMAIN="example.com"
OLD_IP=$(dig $DOMAIN +short)

# Функция проверки изменений
check_dns_changes() {
    NEW_IP=$(dig $DOMAIN +short)
    if [ "$OLD_IP" != "$NEW_IP" ]; then
        echo "🚨 ВНИМАНИЕ: IP адрес изменился!"
        echo "Старый IP: $OLD_IP"
        echo "Новый IP: $NEW_IP"
        OLD_IP=$NEW_IP
    else
        echo "✅ DNS записи без изменений: $NEW_IP"
    fi
}

# Проверять каждые 5 минут
while true; do
    check_dns_changes
    sleep 300
done
```

---

## 🔧 ОПЦИИ ДЛЯ ПРОДВИНУТЫХ СЦЕНАРИЕВ

### **Опции производительности:**
```bash
# Установка таймаута
dig example.com +time=3

# Количество повторных попыток
dig example.com +tries=2

# Размер буфера
dig example.com +bufsize=1024
```

### **Опции отладки:**
```bash
# Подробный вывод
dig example.com +multiline

# Показать все опции
dig -h

# Отладочный режим
dig example.com +debug
```

---

## 🎓 ПРАКТИЧЕСКОЕ ЗАДАНИЕ 5: Полный аудит домена

### **Задача:** Провести полный аудит DNS домена `your-site.com`

```bash
#!/bin/bash
DOMAIN="your-site.com"

echo "=== ПОЛНЫЙ DNS АУДИТ ДОМЕНА: $DOMAIN ==="

# 1. Основные записи
echo "1. ОСНОВНЫЕ ЗАПИСИ:"
dig $DOMAIN A +short
dig $DOMAIN AAAA +short
dig $DOMAIN MX +short
dig $DOMAIN NS +short

# 2. Безопасность
echo -e "\n2. НАСТРОЙКИ БЕЗОПАСНОСТИ:"
dig $DOMAIN TXT | grep -E "(spf|DMARC|DKIM)"
dig _dmarc.$DOMAIN TXT +short

# 3. DNSSEC
echo -e "\n3. ПРОВЕРКА DNSSEC:"
dig $DOMAIN DNSKEY +dnssec | grep -E "(DNSKEY|RRSIG)"

# 4. Время отклика
echo -e "\n4. ПРОИЗВОДИТЕЛЬНОСТЬ:"
time dig $DOMAIN > /dev/null

# 5. Сравнение DNS серверов
echo -e "\n5. СРАВНЕНИЕ DNS СЕРВЕРОВ:"
for dns in "8.8.8.8" "1.1.1.1" "9.9.9.9"; do
    echo "Сервер $dns: $(dig @$dns $DOMAIN +short)"
done
```

---

## 📚 ШПАРГАЛКА ПО КЛАССАМ ОТВЕТОВ DIG

### **Секции вывода dig:**
- **QUESTION** - Ваш запрос
- **ANSWER** - Ответ на запрос
- **AUTHORITY** - Authoritative серверы
- **ADDITIONAL** - Дополнительная информация

### **Управление секциями:**
```bash
# Показать только ответ
dig example.com +noall +answer

# Скрыть вопрос
dig example.com +noquestion

# Показать только authoritative серверы
dig example.com +noall +authority
```

---

## 🎯 ИТОГОВОЕ ЗАДАНИЕ: Расследование инцидента

### **Сценарий:** 
Пользователи жалуются, что их перенаправляет на фишинговый сайт при посещении `bank.com`

### **Ваша задача:** Используя dig, провести расследование

```bash
# 1. Проверить текущий IP
dig bank.com +short

# 2. Сравнить с эталонными DNS
dig @8.8.8.8 bank.com +short
dig @1.1.1.1 bank.com +short

# 3. Проверить DNSSEC
dig bank.com DNSKEY +dnssec

# 4. Проверить исторические записи
dig bank.com A +trace

# 5. Найти различия
diff <(dig @8.8.8.8 bank.com) <(dig @1.1.1.1 bank.com)
```

---

## 💡 СОВЕТЫ ДЛЯ ПОВСЕДНЕВНОЙ РАБОТЫ

1. **Используйте +short** для быстрых проверок
2. **Сохраняйте вывод** для сравнения: `dig example.com > dns_snapshot.txt`
3. **Автоматизируйте проверки** через cron
4. **Используйте +trace** для диагностики сложных проблем
5. **Помните про TTL** - изменения не происходят мгновенно

## ✅ Чеклист освоения dig

- [ ] Базовые запросы A, AAAA, MX, NS записей
- [ ] Работа с разными DNS серверами
- [ ] Использование опций форматирования вывода
- [ ] Трассировка DNS запросов
- [ ] Проверка DNSSEC
- [ ] Написание скриптов для автоматизации
- [ ] Диагностика проблем с DNS

**Dig** - ваш главный инструмент для работы с DNS! 🚀