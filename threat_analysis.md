# Подробное руководство по настройке сканирования уязвимостей на Windows Server 2022

## 🛡️ Обзор инструментов сканирования уязвимостей

### Сравнение инструментов:

| Инструмент | Тип | Стоимость | Особенности |
|------------|-----|-----------|-------------|
| **Nessus** | Коммерческий | Платный | Высокая точность, обширная база |
| **OpenVAS** | Открытый | Бесплатный | Мощная Community-версия |
| **Qualys** | Облачный | Подписка | SaaS решение, не требует установки |

## 🔧 Подготовка Windows Server 2022 к сканированию

### 1. Настройка учетной записи для сканирования

```powershell
# Создание пользователя для сканирования
New-LocalUser -Name "scanner_user" -Description "Account for vulnerability scanning" -NoPassword
Add-LocalGroupMember -Group "Administrators" -Member "scanner_user"

# Или через GUI:
# Панель управления → Учетные записи → Добавить пользователя
```

### 2. Настройка брандмауэра Windows
```powershell
# Включение правил для сканирования
Set-NetFirewallRule -DisplayGroup "File and Printer Sharing" -Enabled True
Set-NetFirewallRule -DisplayGroup "Windows Management Instrumentation (WMI)" -Enabled True

# Открытие портов для различных типов сканирования
New-NetFirewallRule -DisplayName "Vulnerability Scanning" -Direction Inbound -Protocol TCP -LocalPort 135,139,445,3389 -Action Allow
```

### 3. Настройка политик безопасности
```powershell
# Включение аудита для лучшего обнаружения
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable

# Настройка UAC для сканирования
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 0
```

## 📊 Настройка Nessus на Windows Server 2022

### Установка Nessus:

1. **Скачивание:**
   ```powershell
   # Скачать с официального сайта Tenable
   # https://www.tenable.com/downloads/nessus
   ```

2. **Установка:**
   ```powershell
   # Запуск установщика как администратор
   Start-Process -FilePath "Nessus-10.5.0-x64.msi" -Verb RunAs -Wait
   ```

3. **Первоначальная настройка:**
   ```bash
   # После установки открыть в браузере:
   https://localhost:8834
   
   # Выбрать "Nessus Essentials" для бесплатного использования
   # Зарегистрироваться и получить активационный код
   ```

### Создание политики сканирования в Nessus:

```python
# Пример настройки политики через REST API
import requests
import json

nessus_config = {
    "settings": {
        "name": "Windows Server 2022 Comprehensive Scan",
        "description": "Full vulnerability assessment for Windows Server 2022",
        "portscan_range": "default",
        "enabled": True,
        "ping_the_remote_host": "yes",
        "unscanned_closed": "yes",
        "thorough_tests": "yes",
        "http_login_method": "auto"
    },
    "credentials": {
        "windows": {
            "username": "scanner_user",
            "password": "your_secure_password",
            "auth_method": "password"
        }
    }
}

# Сохранение конфигурации
with open('nessus_policy.json', 'w') as f:
    json.dump(nessus_config, f, indent=2)
```

### Настройка сканирования через Web-интерфейс:

1. **Создание нового сканирования:**
   - Advanced Scan → Basic Settings
   - Targets: `localhost` или IP-адрес сервера
   - Port Scan: `Default` (1-65535)

2. **Настройка аутентификации:**
   ```
   Credentials → Windows → Add
   Username: scanner_user
   Password: [пароль]
   Domain: [имя домена или WORKGROUP]
   ```

3. **Настройка плагинов:**
   ```
   Plugins → Family Selection
   ✓ Windows → Microsoft Windows Bulletins
   ✓ General → Service Detection
   ✓ Backdoors → Windows Backdoors
   ✓ Brute Force Attacks → Windows Brute Force
   ```

## 🐧 Настройка OpenVAS для сканирования Windows Server 2022

### Установка OpenVAS на отдельном сервере:

```bash
# Установка на Ubuntu/Debian
sudo apt update
sudo apt install openvas

# Инициализация и настройка
sudo gvm-setup
sudo gvm-start

# Проверка статуса
sudo gvm-check-setup
```

### Настройка сканирования Windows Server 2022 в OpenVAS:

```python
# Конфигурация сканирования через GVM API
openvas_config = {
    "target": {
        "name": "Windows_Server_2022",
        "hosts": ["192.168.1.100"],  # IP вашего сервера
        "port_list": "All TCP and Nmap 5.51 top 1000"
    },
    "scan_config": {
        "name": "Windows_Server_2022_Scan",
        "scanner_type": "OpenVAS Default",
        "target_id": "target_id_here"
    },
    "credentials": {
        "smb": {
            "type": "smb",
            "username": "scanner_user",
            "password": "your_secure_password"
        },
        "ssh": {
            "type": "ssh",
            "username": "administrator",
            "password": "admin_password"
        }
    }
}
```

### Создание задачи сканирования:

```bash
# Через командную строку GVM
gvm-cli --gmp-username admin --gmp-password password socket --xml '
<create_task>
    <name>Windows Server 2022 Vulnerability Scan</name>
    <config id="daba56c8-73ec-11df-a475-002264764cea"/>
    <target id="TARGET_ID"/>
    <scanner id="08b69003-5fc2-4037-a479-93b440211c73"/>
</create_task>'
```

## ☁️ Настройка Qualys для сканирования Windows Server 2022

### Регистрация в Qualys Cloud Platform:

1. **Создание учетной записи:**
   - Посетить https://www.qualys.com/
   - Зарегистрироваться для бесплатной пробной версии

2. **Настройка облачного агента:**
   ```powershell
   # Скачивание и установка Cloud Agent
   Invoke-WebRequest -Uri "https://qualys-agent-download.s3.amazonaws.com/QualysCloudAgent.exe" -OutFile "QualysCloudAgent.exe"
   .\QualysCloudAgent.exe -c -C CustomerID -S ServerID -W Platform -l "C:\Program Files\Qualys\Cloud Agent\qualys-cloud-agent.exe"
   ```

### Конфигурация сканирования в Qualys:

```python
qualys_config = {
    "scan": {
        "title": "Windows Server 2022 Internal Scan",
        "target": {
            "ips": ["192.168.1.100"],
            "asset_groups": ["Windows Servers"]
        },
        "option_profile": {
            "name": "Windows Server 2022 Profile",
            "options": {
                "scan_type": "Internal",
                "authenticated_scan": True,
                "windows_auth": True,
                "performance_settings": "Normal"
            }
        },
        "schedule": {
            "type": "Recurring",
            "frequency": "Weekly",
            "day_of_week": "Sunday",
            "time": "02:00"
        }
    },
    "authentication": {
        "windows": {
            "username": "scanner_user",
            "password": "encrypted_password",
            "domain": "your_domain"
        }
    }
}
```

## 🎯 Практическое задание: Настройка комплексного сканирования

### Шаг 1: Подготовка целевой системы

```powershell
# Скрипт подготовки Windows Server 2022 к сканированию
$ScriptBlock = {
    # Отключение временно антивируса для сканирования
    Set-MpPreference -DisableRealtimeMonitoring $true
    
    # Настройка служб для сканирования
    Get-Service -Name "RemoteRegistry" | Set-Service -StartupType Automatic -Status Running
    Get-Service -Name "Windows Remote Management" | Set-Service -StartupType Automatic -Status Running
    
    # Создание исключений в брандмауэре
    New-NetFirewallRule -DisplayName "VulnScan-TCP" -Direction Inbound -Protocol TCP -LocalPort @(135,139,445,3389,5985,5986) -Action Allow
    New-NetFirewallRule -DisplayName "VulnScan-UDP" -Direction Inbound -Protocol UDP -LocalPort @(137,138) -Action Allow
    
    # Включение необходимых компонентов
    Enable-WindowsOptionalFeature -Online -FeatureName "TelnetClient" -All
}

# Запуск скрипта с правами администратора
Invoke-Command -ScriptBlock $ScriptBlock
```

### Шаг 2: Создание комплексного скана в Nessus

```python
# Python скрипт для автоматизации настройки Nessus
import requests
import json
import base64

class NessusScanner:
    def __init__(self, host, username, password):
        self.host = host
        self.session = requests.Session()
        self.login(username, password)
    
    def login(self, username, password):
        auth = {"username": username, "password": password}
        response = self.session.post(f"{self.host}/session", json=auth, verify=False)
        self.token = response.json()['token']
        self.session.headers.update({'X-Cookie': f'token={self.token}'})
    
    def create_scan(self, config):
        response = self.session.post(f"{self.host}/scans", json=config)
        return response.json()
    
    def start_scan(self, scan_id):
        response = self.session.post(f"{self.host}/scans/{scan_id}/launch")
        return response.json()

# Конфигурация сканирования
scan_config = {
    "uuid": "ad629e16-03b6-8c1d-cef6-ef8c9dd3c658d24bd260ef5f9e66",
    "settings": {
        "name": "Windows Server 2022 Full Audit",
        "description": "Comprehensive vulnerability and compliance scan",
        "text_targets": "192.168.1.100",
        "enabled": True,
        "launch": "ON_DEMAND",
        "folder_id": 1,
        "scanner_id": 1,
        "policy_id": 1,
        "credentials": {
            "Windows": [{
                "username": "scanner_user",
                "password": "secure_password_123",
                "domain": "",
                "auth_method": "password",
                "admin": True
            }]
        }
    }
}

# Создание и запуск сканирования
scanner = NessusScanner("https://localhost:8834", "admin", "nessus_password")
scan = scanner.create_scan(scan_config)
scanner.start_scan(scan['scan']['id'])
```

### Шаг 3: Настройка расписания сканирования

```powershell
# Создание задачи в планировщике Windows для регулярного сканирования
$Action = New-ScheduledTaskAction -Execute "C:\Program Files\Tenable\Nessus\nessuscli.exe" -Argument "scan --target 192.168.1.100 --policy 'Windows Server 2022 Audit'"
$Trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 2am
$Settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries
Register-ScheduledTask -TaskName "Weekly Vulnerability Scan" -Action $Action -Trigger $Trigger -Settings $Settings -User "SYSTEM"
```

### Шаг 4: Мониторинг и анализ результатов

```python
# Скрипт для анализа результатов сканирования
import xml.etree.ElementTree as ET
import pandas as pd

def parse_nessus_results(nessus_file):
    """Парсинг результатов сканирования Nessus"""
    tree = ET.parse(nessus_file)
    root = tree.getroot()
    
    vulnerabilities = []
    for report in root.findall('.//ReportHost'):
        host = report.get('name')
        for item in report.findall('.//ReportItem'):
            vuln = {
                'host': host,
                'port': item.get('port'),
                'service': item.get('svc_name'),
                'protocol': item.get('protocol'),
                'severity': item.get('severity'),
                'plugin_name': item.get('pluginName'),
                'description': item.find('description').text if item.find('description') is not None else '',
                'solution': item.find('solution').text if item.find('solution') is not None else '',
                'risk_factor': item.find('risk_factor').text if item.find('risk_factor') is not None else ''
            }
            vulnerabilities.append(vuln)
    
    return pd.DataFrame(vulnerabilities)

# Анализ результатов
df = parse_nessus_results('scan_results.nessus')
critical_vulns = df[df['severity'] == '4']
print(f"Найдено критических уязвимостей: {len(critical_vulns)}")
```

## 🔒 Безопасность при сканировании

### Рекомендации по безопасности:

```powershell
# Создание специализированной учетной записи только для сканирования
New-LocalUser -Name "vuln_scanner" -Description "Vulnerability scanning account" -NoPassword
Add-LocalGroupMember -Group "Remote Management Users" -Member "vuln_scanner"

# Ограничение прав учетной записи
$acl = Get-Acl "C:\Windows\Temp"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("vuln_scanner", "ReadAndExecute", "Allow")
$acl.SetAccessRule($accessRule)
Set-Acl "C:\Windows\Temp" $acl

# Настройка журналирования действий
wevtutil set-log "Microsoft-Windows-PowerShell/Operational" /enabled:true
```

## 📈 Анализ и отчетность

### Генерация отчетов:

```python
def generate_vulnerability_report(scan_results):
    """Генерация комплексного отчета по уязвимостям"""
    report = {
        'summary': {
            'total_vulnerabilities': len(scan_results),
            'critical_count': len(scan_results[scan_results['severity'] == '4']),
            'high_count': len(scan_results[scan_results['severity'] == '3']),
            'medium_count': len(scan_results[scan_results['severity'] == '2']),
            'low_count': len(scan_results[scan_results['severity'] == '1']),
            'scan_date': pd.Timestamp.now().strftime('%Y-%m-%d %H:%M:%S')
        },
        'critical_vulnerabilities': scan_results[scan_results['severity'] == '4'].to_dict('records'),
        'recommendations': generate_recommendations(scan_results)
    }
    
    return report

def generate_recommendations(df):
    """Генерация рекомендаций по исправлению"""
    recommendations = []
    
    # Анализ наиболее распространенных уязвимостей
    common_vulns = df['plugin_name'].value_counts().head(5)
    
    for vuln, count in common_vulns.items():
        recommendations.append({
            'vulnerability': vuln,
            'affected_systems': count,
            'action': get_remediation_action(vuln)
        })
    
    return recommendations
```

# Подробная инструкция по запуску Python скриптов на Windows Server 2022 с нуля

## 🚀 Установка Python на Windows Server 2022

### Способ 1: Установка из Microsoft Store (Рекомендуется для новичков)

1. **Откройте Microsoft Store:**
   ```powershell
   # Через меню Пуск или выполните:
   start ms-windows-store:
   ```

2. **Найдите Python:**
   - В поиске введите "Python"
   - Выберите "Python 3.11" или новее
   - Нажмите "Установить"

### Способ 2: Ручная установка с официального сайта

1. **Скачайте установщик:**
   ```powershell
   # Перейдите на https://python.org/downloads/
   # Или используйте PowerShell для скачивания
   Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.11.0/python-3.11.0-amd64.exe" -OutFile "python-installer.exe"
   ```

2. **Запустите установку:**
   ```powershell
   # Запустите установщик как администратор
   .\python-installer.exe
   ```

3. **Важные настройки при установке:**
   - ✅ **Add Python to PATH** - ВАЖНО!
   - ✅ **Install launcher for all users** 
   - ✅ **Customize installation**
   - В дополнительных опциях:
     - ✅ **Install for all users**
     - ✅ **Add Python to environment variables**

### Способ 3: Установка через Chocolatey (для продвинутых)

1. **Установите Chocolatey:**
   ```powershell
   # Запустите PowerShell от имени администратора
   Set-ExecutionPolicy Bypass -Scope Process -Force
   [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

2. **Установите Python:**
   ```powershell
   choco install python -y
   ```

## 🔧 Проверка установки Python

### Проверка версии Python:
```powershell
# Откройте новое окно PowerShell или Command Prompt
python --version
# Или
python -V
```

### Проверка пути установки:
```powershell
# Найдите где установлен Python
where python
# Или
Get-Command python
```

### Проверка работы интерпретатора:
```powershell
# Запустите интерактивный режим Python
python
```
```python
# В интерактивном режиме выполните:
print("Hello, Windows Server 2022!")
1 + 1
exit()  # для выхода
```

## 📝 Создание первого Python скрипта

### Способ 1: Создание через Блокнот

1. **Создайте файл:**
   ```powershell
   # Создайте папку для скриптов
   mkdir C:\PythonScripts
   cd C:\PythonScripts
   
   # Создайте файл скрипта
   notepad hello_world.py
   ```

2. **Добавьте код:**
   ```python
   # hello_world.py
   print("Привет, Windows Server 2022!")
   print("Это мой первый Python скрипт!")
   
   # Простые вычисления
   a = 5
   b = 3
   result = a + b
   print(f"{a} + {b} = {result}")
   
   # Информация о системе
   import platform
   import os
   
   print(f"ОС: {platform.system()} {platform.release()}")
   print(f"Версия Python: {platform.python_version()}")
   print(f"Текущая директория: {os.getcwd()}")
   ```

### Способ 2: Создание через PowerShell
```powershell
# Создание файла прямо из PowerShell
@'
# system_info.py
import platform
import socket
import datetime

print("=== ИНФОРМАЦИЯ О СИСТЕМЕ ===")
print(f"Имя компьютера: {socket.gethostname()}")
print(f"ОС: {platform.system()} {platform.version()}")
print(f"Архитектура: {platform.architecture()[0]}")
print(f"Текущее время: {datetime.datetime.now()}")
print(f"Пользователь: {os.getenv('USERNAME')}")
'@ | Out-File -FilePath "C:\PythonScripts\system_info.py" -Encoding utf8
```

## 🏃‍♂️ Запуск Python скриптов

### Способ 1: Запуск из командной строки
```cmd
# Откройте Command Prompt
cd C:\PythonScripts
python hello_world.py
```

### Способ 2: Запуск из PowerShell
```powershell
# Откройте PowerShell
cd C:\PythonScripts
python hello_world.py

# Или используйте полный путь
python C:\PythonScripts\hello_world.py
```

### Способ 3: Двойной клик по файлу (после настройки ассоциаций)
```powershell
# Настройте ассоциацию файлов .py с Python
assoc .py=Python.File
ftype Python.File="C:\Python\Python311\python.exe" "%1" %*

# Теперь можно запускать двойным кликом
```

### Способ 4: Запуск с параметрами
```powershell
# Запуск с передачей аргументов
python script.py arg1 arg2

# Запуск в режиме отладки
python -m pdb script.py

# Запуск с оптимизацией
python -O script.py
```

## ⚙️ Настройка окружения для разработки

### Установка и настройка Visual Studio Code

1. **Скачайте и установите VS Code:**
   ```powershell
   # Скачайте с https://code.visualstudio.com/
   # Или используйте Chocolatey
   choco install vscode -y
   ```

2. **Установите расширение Python:**
   - Откройте VS Code
   - Перейдите в Extensions (Ctrl+Shift+X)
   - Найдите "Python" от Microsoft
   - Установите расширение

3. **Настройте рабочую среду:**
   ```json
   // .vscode/settings.json
   {
       "python.pythonPath": "C:\\Python\\Python311\\python.exe",
       "python.terminal.activateEnvironment": true,
       "python.linting.enabled": true,
       "python.formatting.autopep8Path": "autopep8"
   }
   ```

### Создание виртуального окружения
```powershell
# Создайте виртуальное окружение
python -m venv C:\PythonScripts\myenv

# Активируйте виртуальное окружение
C:\PythonScripts\myenv\Scripts\Activate.ps1

# Теперь все пакеты будут устанавливаться в это окружение
pip install requests pandas numpy

# Деактивация окружения
deactivate
```

## 📦 Установка дополнительных пакетов

### Основные команды pip:
```powershell
# Обновление pip
python -m pip install --upgrade pip

# Установка пакетов
pip install requests
pip install pandas numpy matplotlib
pip install flask django

# Установка конкретной версии
pip install requests==2.28.0

# Установка из файла требований
pip install -r requirements.txt

# Просмотр установленных пакетов
pip list

# Поиск пакетов
pip search "package name"
```

### Популярные пакеты для системного администрирования:
```powershell
# Пакеты для работы с Windows
pip install pywin32
pip install wmi
pip install psutil
pip install paramiko
pip install pyopenssl

# Для автоматизации
pip install selenium
pip install beautifulsoup4
pip install scrapy
```

## 🛠️ Полезные скрипты для Windows Server 2022

### Скрипт для мониторинга системы:
```python
# system_monitor.py
import psutil
import datetime
import time

def system_monitor():
    print("=== МОНИТОРИНГ СИСТЕМЫ ===")
    
    while True:
        # Загрузка CPU
        cpu_percent = psutil.cpu_percent(interval=1)
        
        # Использование памяти
        memory = psutil.virtual_memory()
        
        # Дисковое пространство
        disk = psutil.disk_usage('/')
        
        # Сетевой трафик
        network = psutil.net_io_counters()
        
        print(f"\nВремя: {datetime.datetime.now()}")
        print(f"CPU: {cpu_percent}%")
        print(f"Память: {memory.percent}% использовано")
        print(f"Диск C: {disk.percent}% заполнено")
        print(f"Сеть: Отправлено {network.bytes_sent} байт, Получено {network.bytes_recv} байт")
        
        time.sleep(5)  # Пауза 5 секунд

if __name__ == "__main__":
    system_monitor()
```

### Скрипт для управления службами:
```python
# service_manager.py
import win32service
import win32serviceutil
import subprocess

def list_services():
    """Список всех служб Windows"""
    cmd = 'sc query'
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    print(result.stdout)

def service_control(service_name, action):
    """Управление службой"""
    try:
        if action == 'start':
            win32serviceutil.StartService(service_name)
        elif action == 'stop':
            win32serviceutil.StopService(service_name)
        elif action == 'restart':
            win32serviceutil.RestartService(service_name)
        print(f"Служба {service_name} {action} успешно")
    except Exception as e:
        print(f"Ошибка: {e}")

# Пример использования
if __name__ == "__main__":
    list_services()
    # service_control('Spooler', 'stop')
```

## 🔒 Настройка безопасности для выполнения скриптов

### Разрешение выполнения скриптов в PowerShell:
```powershell
# Проверка текущей политики выполнения
Get-ExecutionPolicy

# Установка политики выполнения (требует админ прав)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Или для текущей сессии
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

### Создание ярлыка для быстрого запуска:
```powershell
# Создание ярлыка для PowerShell с Python
$WshShell = New-Object -comObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$Home\Desktop\PythonPS.lnk")
$Shortcut.TargetPath = "powershell.exe"
$Shortcut.Arguments = "-NoExit -Command `"cd C:\PythonScripts; python`""
$Shortcut.Save()
```

## 🐛 Отладка и решение проблем

### Common Issues and Solutions:

1. **Python не найден в PATH:**
   ```powershell
   # Добавление Python в PATH вручную
   [Environment]::SetEnvironmentVariable(
       "PATH", 
       "$env:PATH;C:\Python\Python311\;C:\Python\Python311\Scripts", 
       "User"
   )
   ```

2. **Ошибка кодировки:**
   ```python
   # Добавьте в начало скрипта
   # -*- coding: utf-8 -*-
   import sys
   sys.stdout.reconfigure(encoding='utf-8')
   ```

3. **Права доступа:**
   ```powershell
   # Запуск PowerShell от имени администратора
   # Или настройка прав через:
   icacls "C:\PythonScripts" /grant "Users:(OI)(CI)RX"
   ```

### Скрипт для диагностики проблем:
```python
# diagnostics.py
import sys
import os

def python_diagnostics():
    print("=== ДИАГНОСТИКА PYTHON ===")
    print(f"Python версия: {sys.version}")
    print(f"Python путь: {sys.executable}")
    print(f"Кодировка: {sys.getdefaultencoding()}")
    print(f"PATH: {os.environ.get('PATH', 'Не найден')}")
    print(f"Текущая директория: {os.getcwd()}")
    
    # Проверка основных модулей
    try:
        import psutil
        print("✓ psutil доступен")
    except ImportError:
        print("✗ psutil не установлен")
    
    try:
        import requests
        print("✓ requests доступен")
    except ImportError:
        print("✗ requests не установлен")

if __name__ == "__main__":
    python_diagnostics()
```

## 📋 Чеклист 

- [ ] Установлен Python 3.11+
- [ ] Python добавлен в PATH
- [ ] Проверена работа `python --version`
- [ ] Создана папка для скриптов
- [ ] Написан и запущен первый скрипт
- [ ] Установлен VS Code с расширением Python
- [ ] Настроено виртуальное окружение
- [ ] Установлены необходимые пакеты
- [ ] Настроена политика выполнения PowerShell


