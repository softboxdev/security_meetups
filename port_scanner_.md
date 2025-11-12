# Подробная инструкция по сканированию портов и изучению структуры сети на Windows Server 2022

## 🎯 Введение для новичков

### Что такое сканирование портов?
Представьте, что ваш сервер - это здание с множеством дверей (портов). Сканирование портов - это проверка, какие двери открыты и что за ними находится.

### Что такое изучение структуры сети?
Это создание "карты" вашей сети - какие устройства подключены, как они связаны между собой.

## 🔧 Подготовка к сканированию

### Установка необходимых инструментов

#### 1. Установка Python (если не установлен)
```powershell
# Скачайте с официального сайта python.org
# Или используйте winget (встроен в Windows 11/Server 2022)
winget install Python.Python.3.11
```

#### 2. Установка необходимых Python пакетов
```powershell
# Откройте PowerShell как администратор
pip install python-nmap
pip install scapy
pip install psutil
pip install networkx
pip install matplotlib
```

#### 3. Установка Nmap (основной инструмент сканирования)
```powershell
# Способ 1: через официальный сайт
# Скачайте с https://nmap.org/download.html

# Способ 2: через Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install nmap -y
```

## 📡 Сканирование открытых портов

### Метод 1: Использование встроенных средств Windows

#### PowerShell сканирование портов:
```powershell
# Простое сканирование одного порта
function Test-Port($computer, $port) {
    try {
        $tcp = New-Object System.Net.Sockets.TcpClient
        $tcp.Connect($computer, $port)
        $tcp.Close()
        return $true
    }
    catch {
        return $false
    }
}

# Сканирование диапазона портов
function Scan-Ports($computer, $startPort, $endPort) {
    Write-Host "Сканирование портов на $computer..." -ForegroundColor Green
    
    for ($port = $startPort; $port -le $endPort; $port++) {
        if (Test-Port $computer $port) {
            Write-Host "Порт $port : ОТКРЫТ" -ForegroundColor Green
        }
    }
}

# Использование:
Scan-Ports "localhost" 1 1000
Scan-Ports "192.168.1.1" 20 443
```

#### Командлет Test-NetConnection:
```powershell
# Проверка конкретного порта
Test-NetConnection -ComputerName "google.com" -Port 80
Test-NetConnection -ComputerName "192.168.1.1" -Port 22

# Сканирование нескольких портов
$ports = @(21, 22, 23, 25, 53, 80, 110, 143, 443, 993, 995, 3389)
foreach ($port in $ports) {
    $result = Test-NetConnection -ComputerName "localhost" -Port $port -InformationLevel Quiet
    if ($result) {
        Write-Host "Порт $port : ОТКРЫТ" -ForegroundColor Green
    }
}
```

### Метод 2: Использование Nmap

#### Базовые команды Nmap:
```powershell
# Сканирование одного хоста
nmap 192.168.1.1

# Сканирование диапазона портов
nmap -p 1-1000 192.168.1.1

# Быстрое сканирование (только основные порты)
nmap -F 192.168.1.1

# Определение версий служб
nmap -sV 192.168.1.1

# Агрессивное сканирование (больше информации)
nmap -A 192.168.1.1

# Сканирование подсети
nmap 192.168.1.0/24

# Сохранение результатов в файл
nmap -oN scan_results.txt 192.168.1.1
```

### Метод 3: Python скрипт для сканирования портов

```python
# port_scanner.py
import socket
import threading
from datetime import datetime

class PortScanner:
    def __init__(self, target, start_port=1, end_port=1000, max_threads=100):
        self.target = target
        self.start_port = start_port
        self.end_port = end_port
        self.max_threads = max_threads
        self.open_ports = []
        self.lock = threading.Lock()
    
    def scan_port(self, port):
        """Сканирование одного порта"""
        try:
            # Создаем сокет
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(1)
            
            # Пытаемся подключиться
            result = sock.connect_ex((self.target, port))
            
            if result == 0:
                with self.lock:
                    self.open_ports.append(port)
                    print(f"Порт {port}: ОТКРЫТ")
            
            sock.close()
            
        except Exception as e:
            pass
    
    def get_service_name(self, port):
        """Получение имени службы для порта"""
        try:
            return socket.getservbyport(port)
        except:
            return "неизвестно"
    
    def run_scan(self):
        """Запуск сканирования"""
        print(f"Начало сканирования {self.target}")
        print(f"Диапазон портов: {self.start_port}-{self.end_port}")
        print("-" * 50)
        
        start_time = datetime.now()
        threads = []
        
        # Создаем потоки для сканирования
        for port in range(self.start_port, self.end_port + 1):
            thread = threading.Thread(target=self.scan_port, args=(port,))
            threads.append(thread)
            thread.start()
            
            # Ограничиваем количество одновременно работающих потоков
            if len(threads) >= self.max_threads:
                for t in threads:
                    t.join()
                threads = []
        
        # Ждем завершения оставшихся потоков
        for thread in threads:
            thread.join()
        
        end_time = datetime.now()
        
        # Вывод результатов
        print("\n" + "=" * 50)
        print("РЕЗУЛЬТАТЫ СКАНИРОВАНИЯ")
        print("=" * 50)
        
        if self.open_ports:
            self.open_ports.sort()
            for port in self.open_ports:
                service = self.get_service_name(port)
                print(f"Порт {port}/tcp : ОТКРЫТ - {service}")
        else:
            print("Открытые порты не найдены")
        
        print(f"\nВремя сканирования: {end_time - start_time}")

# Использование сканера
if __name__ == "__main__":
    # Сканирование localhost
    scanner = PortScanner("localhost", 1, 1000)
    scanner.run_scan()
    
    # Сканирование удаленного хоста
    # scanner = PortScanner("192.168.1.1", 20, 443)
    # scanner.run_scan()
```

## 🗺️ Изучение структуры сети

### Метод 1: Встроенные команды Windows

#### Получение информации о сети:
```powershell
# Информация о сетевых интерфейсах
ipconfig /all
Get-NetIPConfiguration

# Таблица маршрутизации
route print
Get-NetRoute

# ARP таблица (соответствие IP-MAC адресов)
arp -a
Get-NetNeighbor

# Статистика сети
netstat -an
Get-NetTCPConnection

# DNS информация
nslookup google.com
Resolve-DnsName google.com
```

#### Скрипт для сбора сетевой информации:
```powershell
# network_info.ps1
Write-Host "=== ИНФОРМАЦИЯ О СЕТИ ===" -ForegroundColor Cyan

# IP конфигурация
Write-Host "`nIP Конфигурация:" -ForegroundColor Yellow
ipconfig | Select-String -Pattern "(IPv4|Default Gateway|Subnet Mask)"

# Сетевые подключения
Write-Host "`nАктивные TCP подключения:" -ForegroundColor Yellow
netstat -an | Select-String "ESTABLISHED"

# ARP таблица
Write-Host "`nARP Таблица:" -ForegroundColor Yellow
arp -a

# Маршруты
Write-Host "`nТаблица маршрутизации:" -ForegroundColor Yellow
route print | Select-String "Network Destination" -Context 0,10
```

### Метод 2: Python скрипт для анализа сети

```python
# network_analyzer.py
import socket
import subprocess
import platform
import re
from datetime import datetime

class NetworkAnalyzer:
    def __init__(self):
        self.system_info = {}
        self.network_info = {}
    
    def get_system_info(self):
        """Получение информации о системе"""
        self.system_info = {
            'hostname': socket.gethostname(),
            'os': platform.system() + " " + platform.release(),
            'architecture': platform.architecture()[0]
        }
        return self.system_info
    
    def get_ip_info(self):
        """Получение IP информации"""
        try:
            # Получаем локальный IP
            hostname = socket.gethostname()
            local_ip = socket.gethostbyname(hostname)
            
            # Получаем внешний IP (через внешний сервис)
            external_ip = self.get_external_ip()
            
            self.network_info['local_ip'] = local_ip
            self.network_info['external_ip'] = external_ip
            
            return {
                'local_ip': local_ip,
                'external_ip': external_ip
            }
        except Exception as e:
            return {'error': str(e)}
    
    def get_external_ip(self):
        """Получение внешнего IP адреса"""
        try:
            with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
                s.connect(("8.8.8.8", 80))
                return s.getsockname()[0]
        except:
            try:
                import urllib.request
                return urllib.request.urlopen('https://api.ipify.org').read().decode('utf8')
            except:
                return "Не удалось определить"
    
    def scan_local_network(self, network_range="192.168.1.0/24"):
        """Сканирование локальной сети"""
        print(f"Сканирование сети: {network_range}")
        print("-" * 50)
        
        active_hosts = []
        
        # Простое сканирование пингом
        if network_range == "192.168.1.0/24":
            base_ip = "192.168.1."
        else:
            base_ip = network_range.split('.')[0] + "." + network_range.split('.')[1] + "." + network_range.split('.')[2] + "."
        
        for i in range(1, 255):
            ip = base_ip + str(i)
            try:
                # Пинг хоста
                param = '-n' if platform.system().lower() == 'windows' else '-c'
                command = ['ping', param, '1', '-w', '1000', ip]
                
                result = subprocess.run(command, capture_output=True, text=True)
                
                if "TTL=" in result.stdout or "ttl=" in result.stdout:
                    active_hosts.append(ip)
                    print(f"Найден активный хост: {ip}")
                    
                    # Попытка получить имя хоста
                    try:
                        hostname = socket.gethostbyaddr(ip)[0]
                        print(f"  Имя хоста: {hostname}")
                    except:
                        print(f"  Имя хоста: не удалось определить")
                    
            except Exception as e:
                pass
        
        return active_hosts
    
    def get_network_connections(self):
        """Получение активных сетевых подключений"""
        try:
            if platform.system().lower() == 'windows':
                result = subprocess.run(['netstat', '-an'], capture_output=True, text=True)
                connections = []
                
                for line in result.stdout.split('\n'):
                    if 'ESTABLISHED' in line or 'LISTENING' in line:
                        connections.append(line.strip())
                
                return connections
            else:
                return ["Функция доступна только на Windows"]
        except Exception as e:
            return [f"Ошибка: {str(e)}"]
    
    def generate_report(self):
        """Генерация отчета о сети"""
        print("=" * 60)
        print("ОТЧЕТ О СЕТЕВОЙ СТРУКТУРЕ")
        print("=" * 60)
        
        # Системная информация
        print("\n1. СИСТЕМНАЯ ИНФОРМАЦИЯ:")
        sys_info = self.get_system_info()
        for key, value in sys_info.items():
            print(f"   {key}: {value}")
        
        # IP информация
        print("\n2. IP АДРЕСА:")
        ip_info = self.get_ip_info()
        for key, value in ip_info.items():
            print(f"   {key}: {value}")
        
        # Локальная сеть
        print("\n3. СКАНИРОВАНИЕ ЛОКАЛЬНОЙ СЕТИ:")
        active_hosts = self.scan_local_network()
        if active_hosts:
            print(f"   Найдено активных хостов: {len(active_hosts)}")
            for host in active_hosts:
                print(f"   - {host}")
        else:
            print("   Активные хосты не найдены")
        
        # Сетевые подключения
        print("\n4. АКТИВНЫЕ СЕТЕВЫЕ ПОДКЛЮЧЕНИЯ:")
        connections = self.get_network_connections()[:10]  # Показываем первые 10
        for conn in connections:
            print(f"   {conn}")

# Запуск анализатора
if __name__ == "__main__":
    analyzer = NetworkAnalyzer()
    analyzer.generate_report()
```

### Метод 3: Расширенное сканирование с Nmap

```powershell
# Создание полной карты сети
# Сканирование всей подсети с определением ОС
nmap -O 192.168.1.0/24

# Обнаружение активных хостов (без сканирования портов)
nmap -sn 192.168.1.0/24

# Детальное сканирование с определением устройств
nmap -A -T4 192.168.1.0/24

# Сканирование с выводом в удобном формате
nmap -sP 192.168.1.0/24 | Select-String "Nmap scan report"

# Сохранение в XML для последующего анализа
nmap -oX network_scan.xml 192.168.1.0/24
```

## 🛠️ Практические задания для закрепления

### Задание 1: Базовое сканирование
```powershell
# 1. Сканируйте свой собственный сервер
.\port_scanner.py localhost 1 100

# 2. Проверьте основные порты на удаленном хосте
$target = "8.8.8.8"  # Google DNS
21, 22, 53, 80, 443 | ForEach-Object {
    if (Test-NetConnection -ComputerName $target -Port $_ -InformationLevel Quiet) {
        Write-Host "Порт $_ открыт на $target" -ForegroundColor Green
    }
}
```

### Задание 2: Анализ локальной сети
```python
# Создайте карту вашей локальной сети
analyzer = NetworkAnalyzer()

# Определите диапазон вашей сети
# (посмотрите через ipconfig)
your_network = "192.168.1.0/24"  # замените на ваш диапазон

# Просканируйте сеть
hosts = analyzer.scan_local_network(your_network)

print(f"В вашей сети найдено {len(hosts)} устройств")
```

### Задание 3: Мониторинг сетевой активности
```python
# network_monitor.py
import time
import psutil
from datetime import datetime

def monitor_network():
    print("Мониторинг сетевой активности...")
    print("Нажмите Ctrl+C для остановки")
    
    old_stats = psutil.net_io_counters()
    
    try:
        while True:
            time.sleep(5)
            new_stats = psutil.net_io_counters()
            
            sent = new_stats.bytes_sent - old_stats.bytes_sent
            recv = new_stats.bytes_recv - old_stats.bytes_recv
            
            print(f"{datetime.now().strftime('%H:%M:%S')} - "
                  f"Отправлено: {sent/1024:.1f} KB, "
                  f"Получено: {recv/1024:.1f} KB")
            
            old_stats = new_stats
            
    except KeyboardInterrupt:
        print("Мониторинг остановлен")

if __name__ == "__main__":
    monitor_network()
```

## 📊 Визуализация результатов

### Создание простой сетевой карты:
```python
# network_map.py
import matplotlib.pyplot as plt
import networkx as nx

def create_network_map(hosts):
    """Создание визуальной карты сети"""
    G = nx.Graph()
    
    # Добавляем узлы (хосты)
    for host in hosts:
        G.add_node(host, label=host)
    
    # Создаем связи (в реальной сети нужно определить реальные связи)
    # Здесь просто для примера
    for i in range(len(hosts) - 1):
        G.add_edge(hosts[i], hosts[i + 1])
    
    # Рисуем граф
    plt.figure(figsize=(12, 8))
    pos = nx.spring_layout(G)
    nx.draw(G, pos, with_labels=True, node_color='lightblue', 
            node_size=2000, font_size=10, font_weight='bold')
    
    plt.title("Карта сети")
    plt.savefig('network_map.png')
    plt.show()

# Использование
hosts = ['192.168.1.1', '192.168.1.2', '192.168.1.100', '192.168.1.101']
create_network_map(hosts)
```

## 🔒 Меры безопасности при сканировании

### Важные предупреждения:
```python
# security_warning.py
def security_warnings():
    warnings = [
        "⚠️  СКАНИРУЙТЕ ТОЛЬКО СВОИ СЕТИ И СЕРВЕРА",
        "⚠️  Не сканируйте сети без разрешения",
        "⚠️  Сканирование чужих сетей может быть незаконным",
        "⚠️  Используйте эти инструменты только для защиты своих систем",
        "⚠️  Настройте брандмауэр перед сканированием"
    ]
    
    for warning in warnings:
        print(warning)

security_warnings()
```

## 📋 Чеклист 

- [ ] Установлен Python и необходимые пакеты
- [ ] Установлен Nmap
- [ ] Проверена работа базовых команд PowerShell
- [ ] Просканированы порты localhost
- [ ] Получена информация о сетевых интерфейсах
- [ ] Просканирована локальная сеть
- [ ] Создан отчет о сетевой структуре
- [ ] Поняты основные принципы сетевой безопасности

