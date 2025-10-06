Давайте разберем реальный пример передачи пакета по стеку TCP/IP с детальным описанием каждого пакета. Рассмотрим сценарий: **ваш компьютер (192.168.1.100) запрашивает веб-страницу с сервера 93.184.216.34 (example.com).**

---

## **🎯 ПОДРОБНЫЙ РАЗБОР ПЕРЕДАЧИ ПАКЕТА**

### **Этап 1: Подготовка - DNS запрос (UDP)**

**Перед HTTP-запросом нужно преобразовать доменное имя в IP-адрес.**

#### **Пакет 1.1: DNS запрос от клиента к DNS-серверу**

**Прикладной уровень (Application):**
```
DNS Query:
Question: example.com, Type: A, Class: IN
```

**Транспортный уровень (Transport) - UDP:**
```
Source Port: 54321      (случайный высокий порт клиента)
Destination Port: 53    (стандартный порт DNS)
Length: 64              (длина UDP-сегмента)
Checksum: 0xABCD
```

**Сетевой уровень (Internet) - IP:**
```
Version: 4, IHL: 5, Total Length: 84
Identification: 0x1234, Flags: DF, TTL: 64
Protocol: UDP (17)
Source IP: 192.168.1.100
Destination IP: 8.8.8.8    (DNS-сервер Google)
```

**Канальный уровень (Link) - Ethernet:**
```
Destination MAC: 00-1A-2B-3C-4D-5E  (MAC маршрутизатора)
Source MAC: 00-A1-B2-C3-D4-E5       (MAC клиента)
EtherType: 0x0800 (IPv4)
Data: [IP пакет с DNS запросом]
FCS: 0x12345678
```

#### **Пакет 1.2: DNS ответ от сервера к клиенту**

**Прикладной уровень (Application):**
```
DNS Response:
Answer: example.com, Type: A, Class: IN, Address: 93.184.216.34
```

**Транспортный уровень (Transport) - UDP:**
```
Source Port: 53         (порт DNS сервера)
Destination Port: 54321 (порт клиента)
Length: 80
Checksum: 0xDCBA
```

**Сетевой уровень (Internet) - IP:**
```
Source IP: 8.8.8.8
Destination IP: 192.168.1.100
Protocol: UDP (17), TTL: 56
```

**Канальный уровень (Link) - Ethernet:**
```
Destination MAC: 00-A1-B2-C3-D4-E5  (MAC клиента)
Source MAC: 00-1A-2B-3C-4D-5E       (MAC маршрутизатора)
```

---

### **Этап 2: Установление TCP соединения (Three-Way Handshake)**

#### **Пакет 2.1: SYN от клиента к серверу**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555          (случайный порт клиента)
Destination Port: 80        (HTTP порт)
Sequence Number: 1000       (ISN - Initial Sequence Number)
Acknowledgment Number: 0    (пока нет данных для подтверждения)
Header Length: 20 bytes
Flags: SYN ( Synchronize )
Window Size: 65535          (размер окна клиента)
Checksum: 0x1234
Options: MSS=1460, SACK Permitted
```

**Сетевой уровень (Internet) - IP:**
```
Source IP: 192.168.1.100
Destination IP: 93.184.216.34
Protocol: TCP (6), TTL: 64
Identification: 0x1235
```

**Канальный уровень (Link) - Ethernet:**
```
Destination MAC: 00-1A-2B-3C-4D-5E  (шлюз)
Source MAC: 00-A1-B2-C3-D4-E5
```

#### **Пакет 2.2: SYN-ACK от сервера к клиенту**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 80             (HTTP порт сервера)
Destination Port: 55555     (порт клиента)
Sequence Number: 5000       (ISN сервера)
Acknowledgment Number: 1001 (ACK клиентского SYN)
Header Length: 20 bytes
Flags: SYN + ACK            (Synchronize + Acknowledgment)
Window Size: 8192           (размер окна сервера)
Checksum: 0x5678
Options: MSS=1460
```

**Сетевой уровень (Internet) - IP:**
```
Source IP: 93.184.216.34
Destination IP: 192.168.1.100
Protocol: TCP (6), TTL: 54
```

#### **Пакет 2.3: ACK от клиента к серверу**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555
Destination Port: 80
Sequence Number: 1001       (первый байт данных)
Acknowledgment Number: 5001 (ACK серверного SYN)
Header Length: 20 bytes
Flags: ACK                  (Acknowledgment)
Window Size: 65535
Checksum: 0x9ABC
```

**Соединение установлено! Теперь можно передавать данные.**

---

### **Этап 3: Передача HTTP запроса**

#### **Пакет 3.1: HTTP GET запрос**

**Прикладной уровень (Application) - HTTP:**
```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
```

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555
Destination Port: 80
Sequence Number: 1001       (начало данных)
Acknowledgment Number: 5001 (подтверждение полученных данных)
Header Length: 20 bytes
Flags: PSH + ACK            (Push + Acknowledgment)
Window Size: 65535
Checksum: 0xDEF0
```

**Сетевой уровень (Internet) - IP:**
```
Source IP: 192.168.1.100
Destination IP: 93.184.216.34
Protocol: TCP (6), TTL: 64
Total Length: 415           (размер всего IP-пакета)
```

**Канальный уровень (Link) - Ethernet:**
```
Destination MAC: 00-1A-2B-3C-4D-5E
Source MAC: 00-A1-B2-C3-D4-E5
```

---

### **Этап 4: Получение HTTP ответа**

#### **Пакет 4.1: TCP ACK на получение данных**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555
Destination Port: 80
Sequence Number: 1416       (1001 + 415 байт данных)
Acknowledgment Number: 6501 (подтверждение получения данных)
Flags: ACK
Window Size: 65535
```

#### **Пакет 4.2: HTTP ответ от сервера**

**Прикладной уровень (Application) - HTTP:**
```
HTTP/1.1 200 OK
Date: Mon, 23 May 2022 10:15:30 GMT
Server: Apache/2.4.41
Content-Type: text/html; charset=UTF-8
Content-Length: 1524
Connection: keep-alive

<!DOCTYPE html>
<html>
<head><title>Example Domain</title></head>
<body>
<h1>Example Domain</h1>
<p>This domain is for use in illustrative examples...</p>
</body>
</html>
```

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 80
Destination Port: 55555
Sequence Number: 5001       (начало данных ответа)
Acknowledgment Number: 1416 (подтверждение полученных данных)
Flags: PSH + ACK
Window Size: 8192
```

**Сетевой уровень (Internet) - IP:**
```
Source IP: 93.184.216.34
Destination IP: 192.168.1.100
Protocol: TCP (6), TTL: 54
Total Length: 1600
```

---

### **Этап 5: Закрытие соединения (Four-Way Handshake)**

#### **Пакет 5.1: FIN от клиента**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555
Destination Port: 80
Sequence Number: 1416
Acknowledgment Number: 8125
Flags: FIN + ACK            (Finish + Acknowledgment)
```

#### **Пакет 5.2: ACK от сервера**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 80
Destination Port: 55555
Sequence Number: 8125
Acknowledgment Number: 1417
Flags: ACK
```

#### **Пакет 5.3: FIN от сервера**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 80
Destination Port: 55555
Sequence Number: 8125
Acknowledgment Number: 1417
Flags: FIN + ACK
```

#### **Пакет 5.4: ACK от клиента**

**Транспортный уровень (Transport) - TCP:**
```
Source Port: 55555
Destination Port: 80
Sequence Number: 1417
Acknowledgment Number: 8126
Flags: ACK
```

**Соединение закрыто!**

---

## **🔍 КЛЮЧЕВЫЕ МОМЕНТЫ ПРОЦЕССА**

### **Инкапсуляция данных в пути:**

```
[HTTP Data]                    ← Прикладной уровень
[TCP Header][HTTP Data]        ← Транспортный уровень  
[IP Header][TCP][HTTP Data]    ← Сетевой уровень
[Ethernet][IP][TCP][HTTP][FCS] ← Канальный уровень
```

### **Изменения в заголовках при маршрутизации:**

**В локальной сети (до маршрутизатора):**
```
Ethernet: SRC MAC=клиента, DST MAC=маршрутизатора
IP: SRC IP=клиента, DST IP=сервера
```

**После маршрутизатора (в интернете):**
```
Ethernet: SRC MAC=маршрутизатора, DST MAC=следующего хопа
IP: SRC IP=клиента, DST IP=сервера (НЕ меняются!)
```

### **Важность TTL (Time To Live):**

Каждый маршрутизатор уменьшает TTL на 1:
- Исходно: TTL=64
- После 1-го маршрутизатора: TTL=63
- После 2-го маршрутизатора: TTL=62
- Если TTL=0 → пакет отбрасывается (защита от бесконечных петель)

### **Контроль потока через Sequence/Ack numbers:**

```
Клиент: SEQ=1001, ACK=5001 (я отправил 1001, ожидаю 5001)
Сервер: SEQ=5001, ACK=1416 (я отправил 5001, получил до 1416)
```

Этот подробный пример показывает, как каждый уровень добавляет свою служебную информацию, превращая простой HTTP-запрос в сложную последовательность сетевых пакетов, которые надежно доставляются через множество промежуточных устройств.