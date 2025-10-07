подробно разберем динамическую маршрутизацию и протоколы RIPv2, RIPng.

---

## **🔄 ДИНАМИЧЕСКАЯ МАРШРУТИЗАЦИЯ - ОСНОВНЫЕ ПРИНЦИПЫ**

### **Что такое динамическая маршрутизация?**

**Аналогия:** Представьте, что вы таксист в большом городе.

- **Статическая маршрутизация:** Вы запомнили все дороги и маршруты
- **Динамическая маршрутизация:** Вы используете GPS-навигатор, который автоматически показывает объезды при пробках

**Определение:** Динамическая маршрутизация — это автоматический обмен информацией о маршрутах между маршрутизаторами. Маршрутизаторы сами узнают о топологии сети и выбирают оптимальные пути.

### **Преимущества динамической маршрутизации:**

| Преимущество | Объяснение |
|-------------|------------|
| **Автоматизация** | Не нужно вручную прописывать маршруты |
| **Адаптивность** | Автоматический обход сбоев и перегрузок |
| **Масштабируемость** | Подходит для больших и сложных сетей |
| **Отказоустойчивость** | Автоматическое переключение на резервные пути |

### **Классы протоколов динамической маршрутизации:**

1. **Дистанционно-векторные (Distance Vector)**
   - *Пример:* RIP, IGRP
   - Работают по принципу "сплетен" — каждый рассказывает соседям, что знает

2. **Состояния каналов (Link State)**
   - *Пример:* OSPF, IS-IS
   - Строят полную карту сети и самостоятельно вычисляют пути

3. **Гибридные**
   - *Пример:* EIGRP
   - Комбинируют оба подхода

---

## **📡 ПРОТОКОЛ RIPv2 (ROUTING INFORMATION PROTOCOL VERSION 2)**

### **Основные характеристики RIPv2**

**Что такое RIP?**
RIP — это дистанционно-векторный протокол, который использует в качестве метрики **количество переходов (hops)**.

**Ключевые параметры:**
- **Метрика:** Количество хопов (максимум 15)
- **Административное расстояние:** 120
- **Порт:** UDP 520
- **Таймеры:**
  - Update: 30 секунд
  - Invalid: 180 секунд
  - Holddown: 180 секунд
  - Flush: 240 секунд

### **Принцип работы RIPv2**

**Процесс обучения маршрутов:**

1. **Инициализация:** Маршрутизатор отправляет запрос на всех интерфейсах
2. **Обновления:** Каждые 30 секунд отправляет полную таблицу маршрутизации
3. **Соседство:** Получает обновления от соседних маршрутизаторов
4. **Вычисление:** Выбирает маршруты с наименьшим количеством хопов

**Пример распространения маршрута:**
```
Маршрутизатор A: "Я знаю путь к сети 10.1.1.0/24 - 0 hops"
Маршрутизатор B: "Узнал от A: путь к 10.1.1.0/24 - 1 hop"
Маршрутизатор C: "Узнал от B: путь к 10.1.1.0/24 - 2 hops"
```

### **Улучшения RIPv2 по сравнению с RIPv1**

| Характеристика | RIPv1 | RIPv2 |
|----------------|-------|-------|
| **Поддержка VLSM/CIDR** | ❌ Нет | ✅ Да |
| **Аутентификация** | ❌ Нет | ✅ Да |
| **Multicast обновления** | ❌ Broadcast | ✅ Multicast (224.0.0.9) |
| **Маска подсети в обновлениях** | ❌ Нет | ✅ Да |

### **Настройка RIPv2 на Cisco оборудовании**

**Базовая настройка:**
```bash
# Вход в режим конфигурации
Router> enable
Router# configure terminal

# Включение RIP процесса
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary

# Объявление сетей
Router(config-router)# network 192.168.1.0
Router(config-router)# network 192.168.2.0
Router(config-router)# network 10.0.0.0

# Выход и сохранение
Router(config-router)# end
Router# copy running-config startup-config
```

**Расширенная настройка:**
```bash
# Настройка таймеров
Router(config)# router rip
Router(config-router)# timers basic 30 180 180 240

# Настройка пассивного интерфейса (не отправлять обновления)
Router(config-router)# passive-interface gigabitethernet0/0

# Настройка аутентификации
Router(config)# interface gigabitethernet0/1
Router(config-if)# ip rip authentication mode md5
Router(config-if)# ip rip authentication key-chain MY-KEY

Router(config)# key chain MY-KEY
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string MySecretPassword
```

### **Пример топологии сети с RIPv2**

**Сценарий:**
```
    [Router A] -- [Router B] -- [Router C]
    192.168.1.0/24  192.168.2.0/24  192.168.3.0/24
```

**Настройка Router A:**
```bash
RouterA(config)# router rip
RouterA(config-router)# version 2
RouterA(config-router)# no auto-summary
RouterA(config-router)# network 192.168.1.0
RouterA(config-router)# network 192.168.2.0
```

**Настройка Router B:**
```bash
RouterB(config)# router rip
RouterB(config-router)# version 2
RouterB(config-router)# no auto-summary
RouterB(config-router)# network 192.168.2.0
RouterB(config-router)# network 192.168.3.0
```

### **Команды мониторинга и диагностики RIPv2**

**Просмотр таблицы маршрутизации:**
```bash
Router# show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP...
       R    192.168.3.0/24 [120/2] via 192.168.2.2, 00:00:15, GigabitEthernet0/1
```

**Просмотр RIP базы данных:**
```bash
Router# show ip rip database
192.168.1.0/24    auto-summary
192.168.1.0/24    directly connected, GigabitEthernet0/0
192.168.2.0/24    auto-summary
192.168.2.0/24    directly connected, GigabitEthernet0/1
192.168.3.0/24    auto-summary
192.168.3.0/24    [2] via 192.168.2.2, 00:00:12, GigabitEthernet0/1
```

**Отладка RIP:**
```bash
Router# debug ip rip
RIP: received v2 update from 192.168.2.2 on GigabitEthernet0/1
RIP: 192.168.3.0/24 via 0.0.0.0 in 1 hops
```

**Статистика RIP:**
```bash
Router# show ip protocols
Routing Protocol is "rip"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Sending updates every 30 seconds, next due in 18 seconds
  Invalid after 180 seconds, hold down 180, flushed after 240
  Redistributing: rip
  Default version control: send version 2, receive version 2
```

---

## **🌐 ПРОТОКОЛ RIPng (RIP NEXT GENERATION)**

### **Что такое RIPng?**

RIPng — это адаптация протокола RIP для работы с IPv6. Сохраняет основные принципы RIPv2, но работает в IPv6-сетях.

### **Ключевые особенности RIPng**

| Параметр | Значение |
|----------|----------|
| **Транспортный протокол** | UDP (порт 521) |
| **Групповой адрес** | FF02::9 (все RIPng маршрутизаторы) |
| **Метрика** | Количество хопов (максимум 15) |
| **Таймеры** | Аналогичны RIPv2 |

### **Отличия RIPng от RIPv2**

1. **Адресация:** Работает с IPv6-адресами
2. **Транспорт:** Использует IPv6 и UDP порт 521
3. **Групповые рассылки:** FF02::9 вместо 224.0.0.9
4. **Конфигурация:** Настраивается на интерфейсах, а не через network-команды

### **Настройка RIPng на Cisco оборудовании**

**Базовая настройка:**
```bash
# Включение IPv6 маршрутизации
Router(config)# ipv6 unicast-routing

# Создание процесса RIPng
Router(config)# ipv6 router rip MY-RIPNG-PROCESS

# Настройка интерфейсов
Router(config)# interface gigabitethernet0/0
Router(config-if)# ipv6 address 2001:db8:1::1/64
Router(config-if)# ipv6 rip MY-RIPNG-PROCESS enable

Router(config)# interface gigabitethernet0/1
Router(config-if)# ipv6 address 2001:db8:2::1/64
Router(config-if)# ipv6 rip MY-RIPNG-PROCESS enable
```

**Расширенная настройка:**
```bash
# Настройка таймеров
Router(config)# ipv6 router rip MY-RIPNG-PROCESS
Router(config-rtr)# timers 30 180 180 240

# Настройка метрики
Router(config-rtr)# maximum-paths 4
```

### **Пример топологии с RIPng**

**Сценарий:**
```
    [Router X] -- [Router Y] -- [Router Z]
    2001:db8:1::/64  2001:db8:2::/64  2001:db8:3::/64
```

**Настройка Router X:**
```bash
RouterX(config)# ipv6 unicast-routing
RouterX(config)# ipv6 router rip COMPANY-NETWORK
RouterX(config)# interface gigabitethernet0/0
RouterX(config-if)# ipv6 address 2001:db8:1::1/64
RouterX(config-if)# ipv6 rip COMPANY-NETWORK enable
RouterX(config)# interface gigabitethernet0/1
RouterX(config-if)# ipv6 address 2001:db8:2::1/64
RouterX(config-if)# ipv6 rip COMPANY-NETWORK enable
```

### **Команды мониторинга RIPng**

**Просмотр IPv6 таблицы маршрутизации:**
```bash
Router# show ipv6 route
IPv6 Routing Table - default - 5 entries
Codes: C - Connected, L - Local, R - RIP, S - Static...
R   2001:DB8:3::/64 [120/2]
     via FE80::A8BB:CCFF:FE00:200, GigabitEthernet0/1
```

**Просмотр RIPng базы данных:**
```bash
Router# show ipv6 rip database
RIP process "COMPANY-NETWORK", local RIB
 2001:DB8:1::/64, metric 2
    installed GigabitEthernet0/0 00:02:15, expires 00:02:45
```

**Статус RIPng:**
```bash
Router# show ipv6 rip
RIP process "COMPANY-NETWORK", port 521, multicast-group FF02::9
     Administrative distance: 120
     Updates every 30 seconds, expire after 180
     Holddown lasts 180 seconds, garbage collect after 120
```

---

## **⚡ ПРЕИМУЩЕСТВА И НЕДОСТАТКИ RIP**

### **Преимущества RIP:**
- ✅ **Простота** настройки и понимания
- ✅ **Совместимость** между оборудованием разных производителей
- ✅ **Низкие** накладные расходы
- ✅ **Автоматическая** адаптация к изменениям

### **Недостатки RIP:**
- ❌ **Ограничение 15 хопами** — не подходит для больших сетей
- ❌ **Медленная сходимость** — долгое время восстановления после сбоев
- ❌ **Неэффективное использование полосы** — полные обновления каждые 30 секунд
- ❌ **Простая метрика** — не учитывает пропускную способность и задержки

---

## **🛡️ БЕЗОПАСНОСТЬ В RIP**

### **Механизмы аутентификации:**

**RIPv2:**
```bash
# Настройка MD5 аутентификации
Router(config)# key chain RIP-KEY
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string MySecurePassword

Router(config)# interface gigabitethernet0/0
Router(config-if)# ip rip authentication mode md5
Router(config-if)# ip rip authentication key-chain RIP-KEY
```

**RIPng:** Использует встроенные механизмы безопасности IPv6 (IPsec)

---

## **🎯 ПРАКТИЧЕСКИЕ РЕКОМЕНДАЦИИ**

### **Когда использовать RIP:**
- ✅ Маленькие и средние сети (до 15 маршрутизаторов)
- ✅ Однородные сети с одинаковыми каналами
- ✅ Учебные и лабораторные среды
- ✅ Как резервный протокол в гибридных сетях

### **Когда НЕ использовать RIP:**
- ❌ Крупные корпоративные сети
- ❌ Сети с критическими приложениями
- ❌ Сети с разнородными каналами (разная пропускная способность)
- ❌ Сети, требующие быстрой сходимости

### **Альтернативы RIP:**
- **OSPF** — для средних и крупных сетей
- **EIGRP** — для сетей Cisco
- **BGP** — для интернет-маршрутизации

---

## **🔧 ПРАКТИЧЕСКОЕ ЗАДАНИЕ ДЛЯ ЗАКРЕПЛЕНИЯ**

**Сценарий:** Настройте RIPv2 для сети:
```
[Office] -- [Core] -- [Server-Room]
192.168.1.0/24  192.168.2.0/24  192.168.3.0/24
```

**Решение:**
```bash
# На маршрутизаторе Office
Office(config)# router rip
Office(config-router)# version 2
Office(config-router)# no auto-summary
Office(config-router)# network 192.168.1.0
Office(config-router)# network 192.168.2.0

# На маршрутизаторе Core  
Core(config)# router rip
Core(config-router)# version 2
Core(config-router)# no auto-summary
Core(config-router)# network 192.168.2.0
Core(config-router)# network 192.168.3.0
```

**Проверка:**
```bash
Office# show ip route
R    192.168.3.0/24 [120/2] via 192.168.2.2, 00:00:10, GigabitEthernet0/1
```

---

## **🌐 DHCPv4 - ДИНАМИЧЕСКАЯ РАЗДАЧА IP-АДРЕСОВ**

### **ЧТО ТАКОЕ DHCP И ЗАЧЕМ ОН НУЖЕН?**

**Аналогия из жизни:** Представьте, что вы пришли в большой офис в первый день работы.

- **Без DHCP:** Вам нужно самим найти свободный кабинет, узнать его номер, запомнить расположение
- **С DHCP:** Администратор автоматически назначает вам кабинет, выдает ключ и карту прохода

**Определение:** DHCP (Dynamic Host Configuration Protocol) — это протокол, который автоматически назначает IP-адреса и другие сетевые параметры устройствам в сети.

**Проблемы, которые решает DHCP:**
- ❌ Ручное назначение IP-адресов (ошибки, конфликты)
- ❌ Сложность управления большим количеством устройств
- ❌ Невозможность автоматического перераспределения адресов

---

## **🔄 ПРИНЦИП РАБОТЫ DHCPv4 (PROCESS DORA)**

### **4 ЭТАПА ПОЛУЧЕНИЯ IP-АДРЕСА**

Процесс известен как **DORA** (Discover, Offer, Request, Acknowledgment)

#### **Этап 1: DISCOVER - "Ищу DHCP-сервер!"**

**Что происходит:** Клиент подключается к сети и ищет DHCP-сервер.

**Технические детали:**
- **Отправитель:** 0.0.0.0 (клиент еще не имеет IP)
- **Получатель:** 255.255.255.255 (broadcast)
- **Порт:** 67 (сервер), 68 (клиент)
- **Сообщение:** DHCPDISCOVER

```bash
# Пример пакета DHCPDISCOVER
Source MAC: Client-MAC
Source IP: 0.0.0.0
Destination IP: 255.255.255.255
DHCP Message Type: DISCOVER
Client MAC: Client-MAC
```

**Аналогия:** Новый сотрудник кричит: "Есть кто живой? Мне нужен рабочий кабинет!"

#### **Этап 2: OFFER - "Вот тебе адрес!"**

**Что происходит:** DHCP-сервер предлагает клиенту IP-адрес.

**Технические детали:**
- **Отправитель:** IP сервера
- **Получатель:** 255.255.255.255 или Client-MAC
- **Сообщение:** DHCPOFFER

```bash
# Пример пакета DHCPOFFER
Source IP: DHCP-Server-IP
Destination IP: 255.255.255.255
DHCP Message Type: OFFER
Your IP: 192.168.1.100          # Предлагаемый IP
Subnet Mask: 255.255.255.0
Router: 192.168.1.1             # Шлюз по умолчанию
DNS Server: 8.8.8.8             # DNS-сервер
Lease Time: 86400               # Время аренды (секунды)
```

**Аналогия:** Администратор отвечает: "Вот ключ от кабинета 192.168.1.100, шлюз в конце коридора, DNS-справочная на 8.8.8.8"

#### **Этап 3: REQUEST - "Я беру этот адрес!"**

**Что происходит:** Клиент подтверждает, что принимает предложенный адрес.

**Технические детали:**
- **Отправитель:** 0.0.0.0
- **Получатель:** 255.255.255.255
- **Сообщение:** DHCPREQUEST

```bash
# Пример пакета DHCPREQUEST
Source IP: 0.0.0.0
Destination IP: 255.255.255.255
DHCP Message Type: REQUEST
Requested IP: 192.168.1.100     # Запрашиваемый IP
DHCP Server: DHCP-Server-IP     # Выбранный сервер
```

**Аналогия:** Сотрудник говорит: "Отлично! Я беру кабинет 192.168.1.100"

#### **Этап 4: ACKNOWLEDGMENT - "Адрес твой!"**

**Что происходит:** Сервер подтверждает аренду адреса.

**Технические детали:**
- **Отправитель:** IP сервера
- **Получатель:** 255.255.255.255
- **Сообщение:** DHCPACK

```bash
# Пример пакета DHCPACK
Source IP: DHCP-Server-IP
Destination IP: 255.255.255.255
DHCP Message Type: ACK
Your IP: 192.168.1.100
Lease Time: 86400
All Options: Subnet, Gateway, DNS, etc.
```

**Аналогия:** Администратор: "Подтверждаю - кабинет 192.168.1.100 за тобой на 24 часа"

### **ПРОДЛЕНИЕ АРЕНДЫ (RENEWAL PROCESS)**

**T1 (50% времени аренды):** Клиент пытается обновить аренду у оригинального сервера
**T2 (87.5% времени аренды):** Клиент пытается обновить аренду у любого сервера

```bash
# Пример времени аренды 8 дней:
Общая аренда: 8 дней = 691200 секунд
T1 (50%): через 4 дня - запрос к оригинальному серверу
T2 (87.5%): через 7 дней - запрос к любому серверу
```

---

## **⚙️ НАСТРОЙКА DHCP-СЕРВЕРА**

### **НА WINDOWS SERVER**

**Через графический интерфейс:**
1. **Server Manager** → Add Roles → DHCP Server
2. **Создание scope** (области):
   - IP-range: 192.168.1.100 - 192.168.1.200
   - Subnet Mask: 255.255.255.0
   - Lease Duration: 8 дней
3. **Настройка options**:
   - 003 Router: 192.168.1.1
   - 006 DNS: 8.8.8.8, 8.8.4.4
   - 015 Domain Name: company.local

**Через PowerShell:**
```powershell
# Установка роли DHCP
Install-WindowsFeature DHCP -IncludeManagementTools

# Создание scope
Add-DhcpServerV4Scope -Name "Office" `
                      -StartRange 192.168.1.100 `
                      -EndRange 192.168.1.200 `
                      -SubnetMask 255.255.255.0 `
                      -State Active

# Настройка опций
Set-DhcpServerV4OptionValue -ScopeId 192.168.1.0 `
                           -DnsServer 8.8.8.8, 8.8.4.4 `
                           -Router 192.168.1.1
```

### **НА LINUX (ISC DHCP SERVER)**

**Установка:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install isc-dhcp-server

# CentOS/RHEL
sudo yum install dhcp
```

**Конфигурация (/etc/dhcp/dhcpd.conf):**
```bash
# Базовые настройки
option domain-name "company.local";
option domain-name-servers 8.8.8.8, 8.8.4.4;
default-lease-time 600;
max-lease-time 7200;
authoritative;

# Scope для основной сети
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
}

# Резервация для принтера
host printer {
    hardware ethernet 00:11:22:33:44:55;
    fixed-address 192.168.1.50;
}

# Scope для гостевой сети
subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.100 192.168.2.150;
    option routers 192.168.2.1;
}
```

**Запуск службы:**
```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl start isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

### **НА CISCO МАРШРУТИЗАТОРЕ**

**Базовая настройка:**
```bash
# Включение DHCP сервера
Router(config)# ip dhcp pool OFFICE-NETWORK
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
Router(dhcp-config)# domain-name company.local
Router(dhcp-config)# lease 8

# Исключение адресов из раздачи
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.99
Router(config)# ip dhcp excluded-address 192.168.1.201 192.168.1.254

# Резервация для сервера
Router(config)# ip dhcp pool SERVER-RESERVATION
Router(dhcp-config)# host 192.168.1.10 255.255.255.0
Router(dhcp-config)# client-identifier 0100.1111.2222.33
Router(dhcp-config)# client-name Web-Server
```

**Проверка работы:**
```bash
# Просмотр арендованных адресов
Router# show ip dhcp binding
IP address      Client-ID/              Lease expiration    Type
                Hardware address
192.168.1.100   0100.1111.2222.33      Jan 01 2023 10:00 AM Automatic

# Статистика
Router# show ip dhcp server statistics
Memory usage         12546
Address pools        2
Database agents      0
Malformed messages   0
```

---

## **🛡️ АТАКИ НА DHCP И ЗАЩИТА**

### **1. DHCP STARVATION АТАКА**

**Что это:** Злоумышленник истощает пул доступных IP-адресов.

**Как работает:**
```python
# Псевдокод атаки
while True:
    mac = generate_random_mac()
    send_dhcp_discover(mac)
    # Сервер назначает IP для каждого "устройства"
```

**Результат:** Легитимные устройства не могут получить IP-адреса.

**Обнаружение:**
```bash
# На Cisco коммутаторе
Switch# show ip dhcp snooping binding
# Большое количество аренд с разных MAC

# В логах DHCP сервера
"Too many leases for scope 192.168.1.0"
```

**Защита - DHCP Snooping:**
```bash
# Настройка на коммутаторе
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 1
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip dhcp snooping limit rate 10
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# ip dhcp snooping trust  # Для DHCP сервера
```

### **2. ROGUE DHCP SERVER АТАКА**

**Что это:** Злоумышленник запускает неавторизованный DHCP-сервер.

**Как работает:**
```
Легитимный клиент: DHCPDISCOVER
Рogue сервер: DHCPOFFER (злой DNS: 192.168.1.99)
Легитимный сервер: DHCPOFFER (правильный DNS: 8.8.8.8)
Клиент выбирает ПЕРВЫЙ ответ!
```

**Результат:** Клиенты получают ложные сетевые параметры.

**Защита - DHCP Snooping:**
```bash
# Помечаем только доверенные порты как trust
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 1
Switch(config)# interface range gigabitethernet0/1-23
Switch(config-if-range)# no ip dhcp snooping trust  # Клиентские порты
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# ip dhcp snooping trust          # Порты к легитимным серверам
```

### **3. DHCP HIJACKING / MAN-IN-THE-MIDDLE**

**Что это:** Перехват и модификация DHCP-трафика.

**Сценарий атаки:**
1. Атакующий запускает rogue DHCP сервер
2. Назначает себя шлюзом по умолчанию
3. Весь трафик проходит через атакующего

**Защита:**
```bash
# Dynamic ARP Inspection (DAI)
Switch(config)# ip arp inspection vlan 1
Switch(config)# ip arp inspection validate src-mac dst-mac ip

# Source Guard
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip verify source vlan dhcp-snooping
```

### **4. DHCP OPTION MANIPULATION**

**Что это:** Подмена опций DHCP (DNS, шлюз, NTP).

**Защита:**
```bash
# Проверка опций через мониторинг
# Настройка алертов при изменении сетевых параметров
```

---

## **🔧 МОНИТОРИНГ И ДИАГНОСТИКА**

### **КОМАНДЫ ДЛЯ WINDOWS КЛИЕНТА:**

```cmd
# Освободить IP-адрес
ipconfig /release

# Запросить новый IP
ipconfig /renew

# Показать детальную информацию
ipconfig /all

# Просмотр статистики DHCP
netsh dhcp show state
```

### **КОМАНДЫ ДЛЯ LINUX КЛИЕНТА:**

```bash
# Освободить IP-адрес
sudo dhclient -r

# Запросить новый IP
sudo dhclient

# Просмотр логов DHCP
sudo journalctl -u systemd-networkd | grep DHCP
```

### **СКРИПТ МОНИТОРИНГА БЕЗОПАСНОСТИ:**

```bash
#!/bin/bash
# Мониторинг DHCP атак

LOG_FILE="/var/log/dhcp_security.log"
ALERT_EMAIL="admin@company.com"

# Проверка количества аренд
CURRENT_LEASES=$(dhcp-lease-list | wc -l)
MAX_LEASES=50

if [ $CURRENT_LEASES -gt $MAX_LEASES ]; then
    echo "$(date): ALERT - High number of DHCP leases: $CURRENT_LEASES" >> $LOG_FILE
    echo "Possible DHCP starvation attack" | mail -s "DHCP Alert" $ALERT_EMAIL
fi

# Проверка неавторизованных серверов
tcpdump -i eth0 -c 10 port 67 or port 68 | grep -i "dhcp" > /tmp/dhcp_traffic.txt
if grep -q "OFFER" /tmp/dhcp_traffic.txt; then
    UNAUTHORIZED_SERVERS=$(grep "OFFER" /tmp/dhcp_traffic.txt | awk '{print $3}')
    echo "$(date): ALERT - Unauthorized DHCP servers: $UNAUTHORIZED_SERVERS" >> $LOG_FILE
fi
```

---

## **🎯 ЛУЧШИЕ ПРАКТИКИ БЕЗОПАСНОСТИ**

### **1. СЕГМЕНТАЦИЯ СЕТИ**
```bash
# Отдельный VLAN для DHCP сервера
Switch(config)# vlan 100
Switch(config-vlan)# name DHCP-Servers
```

### **2. НАСТРОЙКА DHCP SNOOPING**
```bash
# На всех клиентских коммутаторах
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 1,10,20
Switch(config)# interface range gigabitethernet0/1-23
Switch(config-if-range)# no ip dhcp snooping trust
```

### **3. MAC-ФИЛЬТРАЦИЯ**
```bash
# Ограничение количества MAC-адресов на порту
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport port-security maximum 3
Switch(config-if)# switchport port-security violation restrict
```

### **4. МОНИТОРИНГ И АНАЛИЗ ЛОГОВ**
```bash
# Регулярная проверка логов
sudo tail -f /var/log/syslog | grep dhcp
```

### **5. РЕЗЕРВИРОВАНИЕ DHCP СЕРВЕРОВ**
```bash
# Настройка failover в Windows DHCP
Add-DhcpServerv4Failover -Name "DHCP-Failover" `
                         -ScopeId 192.168.1.0 `
                         -PartnerServer "backup-dhcp.company.local" `
                         -LoadBalancePercent 50
```

---

## **💡 КЛЮЧЕВЫЕ ВЫВОДЫ**

1. **DHCP автоматизирует** управление IP-адресами в сети
2. **Процесс DORA** обеспечивает надежное назначение адресов
3. **Безопасность DHCP** критически важна - используйте DHCP Snooping
4. **Мониторинг** помогает обнаруживать атаки на ранних стадиях
5. **Резервирование** обеспечивает отказоустойчивость службы

Давайте подробно разберем NAT и PAT — фундаментальные технологии, без которых невозможно представить современный интернет.

---

## **🌐 NAT/PAT - ПРЕОБРАЗОВАНИЕ СЕТЕВЫХ АДРЕСОВ**

### **ЧТО ТАКОЕ NAT И ЗАЧЕМ ОН НУЖЕН?**

**Аналогия из жизни:** Представьте большую компанию с внутренней телефонной станцией:

- **Внутренние номера:** 101, 102, 103 (частные адреса)
- **Внешний номер:** +7-495-123-4567 (публичный адрес)
- **Секретарь (NAT)** переключает вызовы: "Вам звонит 101" → "Вам звонит +7-495-123-4567"

**Проблема, которую решает NAT:**
- Нехватка IPv4-адресов (всего ~4.3 миллиарда)
- Безопасность (скрытие внутренней структуры сети)
- Гибкость в изменении сетевой архитектуры

---

## **🔧 NAT (NETWORK ADDRESS TRANSLATION)**

### **ТИПЫ NAT**

#### **1. Статический NAT (Static NAT)**
**Один-к-одному** постоянное соответствие адресов.

**Пример:**
```
Внутренний       Внешний
192.168.1.10  ↔  203.0.113.10
192.168.1.11  ↔  203.0.113.11
```

**Применение:** Для серверов, которые должны быть доступны извне.

#### **2. Динамический NAT (Dynamic NAT)**
**Пул адресов** для временного использования.

**Пример:**
```
Пул внешних адресов: 203.0.113.10-203.0.113.20
Внутренние устройства получают внешние адреса из пула по мере необходимости
```

#### **3. PAT (Port Address Translation) / NAT Overload**
**Один внешний адрес + разные порты** для многих внутренних устройств.

**Это то, что используется в домашних роутерах!**

---

## **🚀 PAT (PORT ADDRESS TRANSLATION) - ДЕТАЛЬНЫЙ РАЗБОР**

### **КАК РАБОТАЕТ PAT?**

**Принцип:** Один публичный IP-адрес + уникальные порты для каждого соединения.

**Пример домашней сети:**
```
Публичный IP: 203.0.113.1
Частные IP: 192.168.1.0/24

Устройства:
- Ноутбук: 192.168.1.100
- Телефон: 192.168.1.101  
- Планшет: 192.168.1.102
```

### **ПРОЦЕСС ТРАНСЛЯЦИИ:**

#### **Исходящее соединение (внутреннее → внешнее)**

**Шаг 1:** Ноутбук открывает веб-страницу
```bash
Источник: 192.168.1.100:54321 → Назначение: 93.184.216.34:80
```

**Шаг 2:** Роутер выполняет PAT
```bash
ДО PAT:   192.168.1.100:54321 → 93.184.216.34:80
ПОСЛЕ PAT: 203.0.113.1:12345  → 93.184.216.34:80

Таблица трансляции роутера:
Внутренний           Внешний           Время
192.168.1.100:54321 → 203.0.113.1:12345  10:30:15
```

**Шаг 3:** Ответ от веб-сервера
```bash
Источник: 93.184.216.34:80 → Назначение: 203.0.113.1:12345
```

**Шаг 4:** Обратная трансляция
```bash
ДО:       93.184.216.34:80 → 203.0.113.1:12345
ПОСЛЕ:    93.184.216.34:80 → 192.168.1.100:54321
```

#### **Одновременные соединения:**

```bash
Таблица PAT роутера:
Устройство           Внешнее отображение     Назначение
192.168.1.100:54321 → 203.0.113.1:12345 → 93.184.216.34:80
192.168.1.101:49152 → 203.0.113.1:12346 → 93.184.216.34:80
192.168.1.102:55555 → 203.0.113.1:12347 → 8.8.8.8:53
```

---

## **⚙️ НАСТРОЙКА NAT/PAT НА CISCO ОБОРУДОВАНИИ**

### **БАЗОВАЯ НАСТРОЙКА PAT (NAT OVERLOAD)**

**Топология:**
```
Внутренняя сеть: 192.168.1.0/24
Внешний интерфейс: GigabitEthernet0/1 (203.0.113.1)
```

**Настройка:**
```bash
# Шаг 1: Определение внутренних и внешних интерфейсов
Router(config)# interface gigabitethernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface gigabitethernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit

# Шаг 2: Создание ACL для определения трафика для трансляции
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255

# Шаг 3: Настройка PAT (использовать внешний IP интерфейса)
Router(config)# ip nat inside source list 1 interface gigabitethernet0/1 overload
```

### **СТАТИЧЕСКИЙ NAT ДЛЯ СЕРВЕРА**

**Для веб-сервера во внутренней сети:**
```bash
# Постоянное соответствие для веб-сервера
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.10

# Или для конкретного порта (Port Forwarding)
Router(config)# ip nat inside source static tcp 192.168.1.10 80 203.0.113.1 80
Router(config)# ip nat inside source static tcp 192.168.1.10 443 203.0.113.1 443
```

### **ДИНАМИЧЕСКИЙ NAT С ПУЛОМ АДРЕСОВ**

```bash
# Создание пула внешних адресов
Router(config)# ip nat pool MY-POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0

# Применение пула к ACL
Router(config)# ip nat inside source list 1 pool MY-POOL
```

---

## **🔍 ПРИМЕРЫ ТРАНСЛЯЦИИ В ДЕТАЛЯХ**

### **ПРИМЕР 1: ВЕБ-СЕРФИНГ**

**Исходный пакет от клиента:**
```
Ethernet: SRC MAC Client, DST MAC Router
IP: SRC 192.168.1.100, DST 93.184.216.34
TCP: SRC Port 54321, DST Port 80
Data: HTTP GET request
```

**После PAT на роутере:**
```
Ethernet: SRC MAC Router, DST MAC Next-Hop
IP: SRC 203.0.113.1, DST 93.184.216.34
TCP: SRC Port 12345, DST Port 80
Data: HTTP GET request
```

**Таблица трансляции:**
```bash
Router# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
tcp 203.0.113.1:12345 192.168.1.100:54321 93.184.216.34:80  93.184.216.34:80
```

### **ПРИМЕР 2: PORT FORWARDING ДЛЯ СЕРВЕРА**

**Входящее соединение из интернета:**
```
IP: SRC 198.51.100.50, DST 203.0.113.1
TCP: SRC Port 49152, DST Port 80
```

**После трансляции на роутере:**
```
IP: SRC 198.51.100.50, DST 192.168.1.10
TCP: SRC Port 49152, DST Port 80
```

---

## **📊 ТИПЫ NAT ПО СТЕПЕНИ ОГРАНИЧЕНИЙ**

### **1. Full Cone NAT**
- **Входящие соединения:** Разрешены с любого внешнего устройства
- **Аналогия:** Дом с открытой дверью "кто угодно может зайти"

### **2. Restricted Cone NAT**
- **Входящие соединения:** Только от адресов, к которым было исходящее соединение
- **Аналогия:** Дом, куда пускают только гостей, которых вы посещали

### **3. Port Restricted Cone NAT**
- **Входящие соединения:** Только с тех же адрес:порт
- **Аналогия:** Дом, куда пускают только конкретных гостей из конкретных мест

### **4. Symmetric NAT**
- **Разные порты** для разных получателей
- **Самый строгий** тип NAT
- **Аналогия:** Для каждого гостя — отдельная дверь

---

## **🛡️ БЕЗОПАСНОСТЬ NAT**

### **Преимущества безопасности:**

1. **Скрытие внутренней сети:** Внешние устройства не видят внутреннюю структуру
2. **Неявный firewall:** Блокируются неинициированные входящие соединения
3. **Изоляция устройств:** Внутренние устройства защищены от прямого доступа

### **Ограничения:**

1. **Проблемы с P2P-приложениями** (Torrent, VoIP, игры)
2. **Сложность настройки серверов** во внутренней сети
3. **Дополнительная задержка** из-за трансляции

---

## **🔧 РЕШЕНИЕ ПРОБЛЕМ С NAT**

### **1. Port Forwarding (Проброс портов)**

**Для игрового сервера:**
```bash
# На Cisco
Router(config)# ip nat inside source static udp 192.168.1.50 27015 203.0.113.1 27015

# В домашних роутерах через веб-интерфейс:
# Внешний порт: 27015 → Внутренний IP: 192.168.1.50:27015
```

### **2. UPnP (Universal Plug and Play)**
- Автоматическая настройка проброса портов
- Используется играми и P2P-приложениями
- **Включение:** `Router(config)# ip nat enable upnp`

### **3. DMZ (Demilitarized Zone)**
- Полный проброс всех портов на одно устройство
- **Опасно!** Устройство полностью открыто в интернет

```bash
# Настройка DMZ
Router(config)# ip nat inside source static 192.168.1.100 203.0.113.1
```

---

## **💻 КОМАНДЫ МОНИТОРИНГА И ДИАГНОСТИКИ**

### **ПРОСМОТР ТАБЛИЦЫ NAT:**
```bash
# Текущие трансляции
Router# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 203.0.113.1:512  192.168.1.100:512  8.8.8.8:512        8.8.8.8:512
tcp  203.0.113.1:12345 192.168.1.100:54321 93.184.216.34:80 93.184.216.34:80

# Детальная информация о конкретной трансляции
Router# show ip nat translations verbose

# Статистика NAT
Router# show ip nat statistics
Total active translations: 3 (0 static, 3 dynamic; 3 extended)
Peak translations: 15, occurred 00:15:25 ago
Outside interfaces:
  GigabitEthernet0/1
Inside interfaces: 
  GigabitEthernet0/0
Hits: 1450  Misses: 5
```

### **ОТЛАДКА NAT:**
```bash
# Включение отладки
Router# debug ip nat
Router# debug ip nat detailed

# Пример вывода отладки
NAT: s=192.168.1.100->203.0.113.1, d=93.184.216.34 [12345]
NAT*: s=93.184.216.34, d=203.0.113.1->192.168.1.100 [12345]
```

### **ОЧИСТКА NAT ТАБЛИЦЫ:**
```bash
# Очистка всех динамических трансляций
Router# clear ip nat translation *

# Очистка конкретной трансляции
Router# clear ip nat translation inside 203.0.113.1 12345
```

---

## **🌐 NAT ДЛЯ IPv6**

### **Нужен ли NAT для IPv6?**
- **НЕТ!** IPv6 имеет достаточно адресов (340 секстиллионов)
- Но используются другие механизмы безопасности:
  - **Stateful firewall**
  - **Privacy extensions**
  - **Unique Local Addresses (ULA)**

### **NAT64** - для взаимодействия IPv6 и IPv4:
```bash
# Префикс NAT64
Router(config)# nat64 prefix stateful 64:ff9b::/96
```

---

## **🎯 ПРАКТИЧЕСКИЕ СЦЕНАРИИ**

### **СЦЕНАРИЙ 1: МАЛЫЙ ОФИС**
```
Внутренняя сеть: 192.168.1.0/24
Публичный IP: 203.0.113.1
Требуется: Веб-сервер доступен извне
```

**Настройка:**
```bash
# PAT для всех устройств
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface gigabitethernet0/1 overload

# Port forwarding для веб-сервера
Router(config)# ip nat inside source static tcp 192.168.1.10 80 203.0.113.1 80
```

### **СЦЕНАРИЙ 2: КОРПОРАТИВНАЯ СЕТЬ**
```
Внутренние сети: 10.1.1.0/24, 10.1.2.0/24
Пул публичных IP: 203.0.113.10-203.0.113.20
Серверы нуждаются в статических сопоставлениях
```

**Настройка:**
```bash
# Динамический NAT для пользователей
Router(config)# ip nat pool INTERNET-POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
Router(config)# access-list 1 permit 10.1.0.0 0.0.255.255
Router(config)# ip nat inside source list 1 pool INTERNET-POOL

# Статический NAT для серверов
Router(config)# ip nat inside source static 10.1.1.10 203.0.113.9
```

---

## **🔍 РЕАЛЬНЫЙ ПРИМЕР ИЗ ЖИЗНИ**

**Домашний роутер:**
```
Публичный IP: 95.165.123.44 (выдан провайдером)
Внутренняя сеть: 192.168.1.0/24

События:
1. Телефон (192.168.1.101:49152) открывает Instagram
   PAT: 192.168.1.101:49152 → 95.165.123.44:54321

2. Ноутбук (192.168.1.100:55555) открывает YouTube  
   PAT: 192.168.1.100:55555 → 95.165.123.44:54322

3. Игровая консоль (192.168.1.50:3074) требует открытый порт
   Port Forwarding: 95.165.123.44:3074 → 192.168.1.50:3074
```

---

## **💡 КЛЮЧЕВЫЕ ВЫВОДЫ**

1. **NAT решает проблему нехватки IPv4-адресов**
2. **PAT (NAT Overload)** — самый распространенный тип
3. **Статический NAT** используется для серверов
4. **Port Forwarding** необходим для входящих соединений
5. **NAT обеспечивает базовую безопасность**
6. **Мониторинг через `show ip nat translations`**

**Формула PAT:** 
```
Один публичный IP + 65535 портов = Множество внутренних устройств
```

