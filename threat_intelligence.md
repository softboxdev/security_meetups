Подробное руководство по работе с Threat Intelligence на Kali Linux с использованием MISP.

## 1. Настройка MISP на Kali Linux

### **Предварительные требования**

```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Проверка версии Python
python3 --version  # Должен быть 3.6+

# Проверка свободного места
df -h  # Требуется минимум 20GB свободного места
```

### **Установка MISP с помощью скрипта**

```bash
# Скачивание официального скрипта установки
wget https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh -O /tmp/INSTALL.sh

# Запуск установки
bash /tmp/INSTALL.sh -A  # -A для автоматической установки

# Или пошаговая установка
bash /tmp/INSTALL.sh -s  # Интерактивный режим
```

### **Ручная установка компонентов**

#### **Установка зависимостей**
```bash
# Базовые зависимости
sudo apt install -y curl gcc git gnupg-agent make python3 python3-pip \
redis-server sudo vim zip

# База данных
sudo apt install -y mariadb-client mariadb-server

# Веб-сервер
sudo apt install -y apache2 apache2-doc apache2-utils libapache2-mod-php7.4

# PHP расширения
sudo apt install -y php7.4 php7.4-cli php7.4-dev php7.4-json php7.4-mysql \
php7.4-opcache php7.4-readline php7.4-xml php7.4-mbstring \
php7.4-curl php7.4-gd php7.4-zip
```

#### **Настройка MySQL/MariaDB**
```bash
# Запуск безопасной настройки
sudo mysql_secure_installation

# Создание базы данных MISP
sudo mysql -u root -p

# В MySQL выполнить:
CREATE DATABASE misp;
GRANT ALL PRIVILEGES ON misp.* TO 'misp'@'localhost' IDENTIFIED BY 'StrongPassword123!';
FLUSH PRIVILEGES;
EXIT;
```

#### **Установка ядра MISP**
```bash
# Создание пользователя MISP
sudo adduser --system --home /var/www/MISP --group misp

# Клонирование репозитория
cd /var/www
sudo git clone https://github.com/MISP/MISP.git
sudo chown -R misp:misp MISP
cd MISP

# Установка зависимостей Python
sudo -H -u misp /var/www/MISP/venv/bin/pip3 install -r requirements.txt

# Настройка конфигурации
sudo cp /var/www/MISP/config/default.php /var/www/MISP/config.php
sudo chown misp:misp /var/www/MISP/config.php
```

### **Настройка веб-сервера**

```bash
# Активация модулей Apache
sudo a2enmod rewrite headers ssl

# Копирование конфигурации MISP
sudo cp /var/www/MISP/INSTALL/apache.24.misp.ssl /etc/apache2/sites-available/misp-ssl.conf

# Настройка SSL (самоподписанный сертификат)
sudo openssl req -newkey rsa:4096 -days 365 -nodes -x509 \
-subj "/C=US/ST=State/L=City/O=Organization/CN=misp.local" \
-keyout /etc/ssl/private/misp.key -out /etc/ssl/private/misp.crt

# Активация сайта
sudo a2ensite misp-ssl.conf
sudo systemctl restart apache2
```

### **Первоначальная настройка MISP**

```bash
# Инициализация базы данных
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "MISP.baseurl" "https://localhost"
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "MISP.redis_host" "127.0.0.1"
sudo -u www-data /var/www/MISP/app/Console/cake database update
sudo -u www-data /var/www/MISP/app/Console/cake userInit -q

# Создание администратора
sudo -u www-data /var/www/MISP/app/Console/cake user -a \
--email admin@misp.local \
--password SuperSecurePassword123! \
--org_id 1 \
--role_id 1
```

### **Проверка установки**

```bash
# Проверка сервисов
sudo systemctl status apache2
sudo systemctl status mysql
sudo systemctl status redis-server

# Проверка доступа через браузер
curl -k https://localhost
```

## 2. Интеграция внешних источников TI

### **Настройка синхронизации с другими экземплярами MISP**

#### **Добавление сервера MISP для синхронизации**
```bash
# Через веб-интерфейс:
# Administration → Server Settings → Synchronization Servers → New Sync Server

# Или через CLI
sudo -u www-data /var/www/MISP/app/Console/cake Server addSyncServer \
--url "https://other-misp.instance" \
--authkey "API_KEY_FROM_OTHER_MISP" \
--push 1 --pull 1
```

#### **Настройка расписания синхронизации**
```bash
# Добавление в cron
sudo crontab -u www-data -e

# Добавить строки:
*/15 * * * * /var/www/MISP/app/Console/cake Server pullAll 1
*/15 * * * * /var/www/MISP/app/Console/cake Server pushAll 1
```

### **Интеграция с открытыми источниками TI**

#### **Настройка feeds в MISP**
```bash
# Просмотр доступных feeds
sudo -u www-data /var/www/MISP/app/Console/cake user -l

# Добавление feed через CLI
sudo -u www-data /var/www/MISP/app/Console/cake Feed add \
--url "https://feeds.feedburner.com/MalwareDomainList" \
--name "Malware Domain List" \
--provider "MalwareDomainList" \
--source_format "misp"
```

#### **Популярные открытые источники для интеграции**

```bash
# CIRCL Taxii Server
sudo -u www-data /var/www/MISP/app/Console/cake Feed add \
--url "https://taxii.circl.lu/taxii-discovery-service" \
--name "CIRCL Taxii" \
--provider "CIRCL" \
--source_format "taxii"

# AlienVault OTX (требуется API ключ)
sudo -u www-data /var/www/MISP/app/Console/cake Feed add \
--url "https://otx.alienvault.com/api/v1/pulses/subscribed" \
--name "AlienVault OTX" \
--provider "AlienVault" \
--source_format "misp" \
--input_source "network" \
--headers '{"X-OTX-API-KEY": "YOUR_API_KEY"}'

# Abuse.ch Feeds
sudo -u www-data /var/www/MISP/app/Console/cake Feed add \
--url "https://urlhaus.abuse.ch/feeds/csv/" \
--name "URLhaus" \
--provider "abuse.ch" \
--source_format "csv"
```

### **Настройка TAXII клиента**

```bash
# Установка Python библиотек
sudo pip3 install cabby stomp-py taxii2-client

# Настройка TAXII клиента через MISP UI
# Administration → Threat Intelligence → TAXII Servers → Add TAXII Server
```

#### **Пример скрипта для TAXII**
```python
#!/usr/bin/env python3
from cabby import create_client
import json

# Настройка TAXII клиента
client = create_client(
    'misp.local',
    use_https=True,
    discovery_path='/taxii-discovery-service'
)

# Аутентификация
client.set_auth(
    username='admin',
    password='password',
    verify_ssl=False
)

# Получение коллекций
collections = client.get_collections()

for collection in collections:
    print(f"Collection: {collection.name}")
    # Получение данных
    content_blocks = client.poll(collection_name=collection.name)
    for block in content_blocks:
        print(block.content)
```

### **Интеграция с коммерческими источниками**

#### **Настройка VirusTotal**
```bash
# Установка VT расширения для MISP
cd /var/www/MISP/app/files/scripts
sudo git clone https://github.com/MISP/misp-modules.git
cd misp-modules
sudo pip3 install -r REQUIREMENTS

# Запуск misp-modules
sudo -u www-data /var/www/MISP/app/files/scripts/misp-modules/misp-modules.py &
```

#### **Конфигурация API ключей**
```bash
# Установка API ключей через CLI
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.CIRCL_username" "your_username"
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.CIRCL_password" "your_password"
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.VirusTotal_apikey" "your_vt_key"
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.OTX_apikey" "your_otx_key"
```

## 3. Создание сигнатуры на основе IoC

### **Типы IoC в MISP**

#### **Основные типы индикаторов:**
- **domain** - Доменные имена
- **ip-src/ip-dst** - IP адреса
- **url** - URL адреса
- **md5/sha1/sha256** - Хеши файлов
- **email-src/email-dst** - Email адреса
- **filename** - Имена файлов

### **Создание события с IoC через веб-интерфейс**

1. **Создание нового события:**
   - Events → Add Event
   - Заполнение метаданных:
     - Distribution: Your Organization Only
     - Threat Level: High
     - Analysis: Completed

2. **Добавление атрибутов:**
   - Add Attribute → Выбор типа индикатора
   - Категория: Network activity, Payload delivery и т.д.
   - Тип: domain, ip-dst, url и т.д.

### **Создание событий через API**

#### **Настройка API ключа**
```bash
# Получение API ключа
sudo -u www-data /var/www/MISP/app/Console/cake user -l

# Или через веб-интерфейс:
# Administration → User Management → Edit User → Auth Key
```

#### **Python скрипт для добавления события**
```python
#!/usr/bin/env python3
import requests
import json
import sys

class MISPClient:
    def __init__(self, base_url, api_key, verify_ssl=False):
        self.base_url = base_url
        self.headers = {
            'Authorization': api_key,
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        }
        self.verify_ssl = verify_ssl
        
    def create_event(self, event_info, attributes):
        """Создание нового события с атрибутами"""
        event_data = {
            "Event": {
                "info": event_info,
                "distribution": "1",  # Your Organization Only
                "threat_level_id": "1",  # High
                "analysis": "2",  # Completed
                "Attribute": attributes
            }
        }
        
        url = f"{self.base_url}/events"
        response = requests.post(
            url,
            headers=self.headers,
            data=json.dumps(event_data),
            verify=self.verify_ssl
        )
        
        return response.json()

# Использование
misp = MISPClient(
    base_url="https://localhost",
    api_key="YOUR_API_KEY"
)

# Создание события с IoC
attributes = [
    {
        "type": "ip-dst",
        "category": "Network activity",
        "value": "192.168.1.100",
        "comment": "C2 server IP"
    },
    {
        "type": "domain",
        "category": "Network activity", 
        "value": "malicious-domain.com",
        "comment": "Phishing domain"
    },
    {
        "type": "md5",
        "category": "Payload delivery",
        "value": "5d41402abc4b2a76b9719d911017c592",
        "comment": "Malware sample"
    }
]

result = misp.create_event(
    "APT29 Campaign Indicators",
    attributes
)
print(json.dumps(result, indent=2))
```

### **Создание сложных сигнатур**

#### **YARA правила через MISP**
```python
# Добавление YARA правила как атрибута
yara_rule = """
rule APT29_Backdoor {
    meta:
        author = "SOC Team"
        description = "Detects APT29 custom backdoor"
        date = "2024-01-01"
    
    strings:
        $str1 = "cmd.exe /c" nocase
        $str2 = { 48 8B 05 ?? ?? ?? ?? 48 85 C0 74 ?? 8B 40 }
        $str3 = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    
    condition:
        any of them and filesize < 500KB
}
"""

attribute = {
    "type": "yara",
    "category": "Payload delivery", 
    "value": yara_rule,
    "comment": "APT29 custom backdoor detection"
}
```

#### **Sigma правила**
```python
sigma_rule = {
    "title": "Suspicious PowerShell Execution",
    "description": "Detects suspicious PowerShell commands",
    "logsource": {
        "product": "windows",
        "service": "powershell"
    },
    "detection": {
        "selection": {
            "CommandLine|contains": [
                "Invoke-Expression",
                "DownloadString",
                "Base64"
            ]
        },
        "condition": "selection"
    },
    "level": "high"
}

attribute = {
    "type": "sigma",
    "category": "Execution",
    "value": json.dumps(sigma_rule),
    "comment": "Suspicious PowerShell detection"
}
```

### **Экспорт сигнатур**

#### **Экспорт в STIX format**
```bash
# Экспорт через API
curl -H "Authorization: YOUR_API_KEY" \
-H "Accept: application/stix+json" \
"https://localhost/events/1" -o event_1.stix.json
```

#### **Экспорт для Suricata**
```bash
# Настройка NIDS экспорта
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.Export_services_enable" true
sudo -u www-data /var/www/MISP/app/Console/cake Admin setSetting "Plugin.Export_suricata_enable" true

# Экспорт через UI: Events → View Event → Export → Suricata
```

#### **Экспорт для Snort**
```python
# Python скрипт для конвертации
def convert_to_snort(misp_event):
    snort_rules = []
    for attribute in misp_event['Attribute']:
        if attribute['type'] in ['ip-src', 'ip-dst']:
            rule = f"alert ip {attribute['value']} any -> any any (msg:\"MISP Event {misp_event['id']}\"; sid:100000{misp_event['id']}; rev:1;)"
            snort_rules.append(rule)
    return snort_rules
```

## 4. Обогащение событий в SIEM с помощью TI

### **Интеграция MISP с Elastic Stack**

#### **Установка MISP модуля для Elastic**
```bash
# Установка необходимых пакетов
sudo pip3 install elasticsearch pytz requests

# Клонирование MISP-to-Elastic скрипта
cd /opt
sudo git clone https://github.com/MISP/misp-modules.git
cd misp-modules/misp_modules/modules/expansion
```

#### **Настройка Elasticsearch connector**
```python
#!/usr/bin/env python3
from pymisp import ExpandedPyMISP
from elasticsearch import Elasticsearch
import json
import time

class MISPElasticConnector:
    def __init__(self, misp_url, misp_key, es_host, es_index):
        self.misp = ExpandedPyMISP(misp_url, misp_key, ssl=False)
        self.es = Elasticsearch([es_host])
        self.es_index = es_index
        
    def enrich_ip(self, ip_address):
        """Обогащение IP адреса данными из MISP"""
        # Поиск в MISP
        result = self.misp.search('attributes', value=ip_address)
        
        enrichment_data = {
            'ip': ip_address,
            'misp_events': [],
            'threat_level': 'low',
            'tags': []
        }
        
        if result:
            for event in result:
                event_info = {
                    'event_id': event['Event']['id'],
                    'info': event['Event']['info'],
                    'threat_level': event['Event']['threat_level_id'],
                    'tags': [tag['name'] for tag in event['Event'].get('Tag', [])]
                }
                enrichment_data['misp_events'].append(event_info)
                
            # Определение максимального уровня угрозы
            threat_levels = [e['threat_level'] for e in enrichment_data['misp_events']]
            enrichment_data['threat_level'] = max(threat_levels) if threat_levels else 'low'
            
        return enrichment_data
    
    def index_to_elastic(self, doc_id, document):
        """Индексация документа в Elasticsearch"""
        self.es.index(
            index=self.es_index,
            id=doc_id,
            body=document
        )

# Использование
connector = MISPElasticConnector(
    misp_url='https://localhost',
    misp_key='YOUR_API_KEY',
    es_host='localhost:9200',
    es_index='misp-enrichment'
)

# Пример обогащения
ip_to_check = '192.168.1.100'
enriched_data = connector.enrich_ip(ip_to_check)
connector.index_to_elastic(ip_to_check, enriched_data)
```

### **Интеграция с Splunk**

#### **Установка MISP App для Splunk**
```bash
# Скачивание MISP App
wget https://github.com/MISP/MISP-Splunk/archive/refs/heads/main.zip -O /tmp/misp-splunk.zip
unzip /tmp/misp-splunk.zip -d /opt/splunk/etc/apps/
```

#### **Splunk search command для MISP**
```python
#!/usr/bin/env python3
import splunk.Intersplunk
import requests
import json

def misp_lookup():
    results, dummyresults, settings = splunk.Intersplunk.getOrganizedResults()
    
    misp_url = "https://localhost"
    api_key = "YOUR_API_KEY"
    headers = {
        'Authorization': api_key,
        'Content-Type': 'application/json'
    }
    
    for result in results:
        # Проверка IP адресов
        if 'src_ip' in result:
            ip = result['src_ip']
            response = requests.post(
                f"{misp_url}/attributes/restSearch",
                headers=headers,
                json={"value": ip},
                verify=False
            )
            
            if response.status_code == 200:
                misp_data = response.json()
                if misp_data.get('response', {}).get('Attribute'):
                    result['misp_threat'] = 'true'
                    result['misp_events'] = len(misp_data['response']['Attribute'])
                else:
                    result['misp_threat'] = 'false'
                    result['misp_events'] = 0
                    
    splunk.Intersplunk.outputResults(results)

if __name__ == '__main__':
    misp_lookup()
```

### **Real-time обогащение с помощью Logstash**

#### **Logstash конфигурация**
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # Обогащение IP адресов через MISP
  if [src_ip] {
    ruby {
      code => '
        require "net/http"
        require "json"
        
        def misp_lookup(ip)
          uri = URI.parse("https://localhost/attributes/restSearch")
          request = Net::HTTP::Post.new(uri)
          request["Authorization"] = "YOUR_API_KEY"
          request["Content-Type"] = "application/json"
          request.body = JSON.dump({"value": ip})
          
          response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true, verify_ssl: false) do |http|
            http.request(request)
          end
          
          JSON.parse(response.body) if response.code == "200"
        end
        
        misp_data = misp_lookup(event.get("src_ip"))
        if misp_data && misp_data["response"] && misp_data["response"]["Attribute"]
          event.set("[misp][threat_level]", "high")
          event.set("[misp][event_count]", misp_data["response"]["Attribute"].length)
          event.set("[misp][events]", misp_data["response"]["Attribute"].map { |a| a["Event"]["info"] })
        else
          event.set("[misp][threat_level]", "low")
        end
      '
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "enriched-logs-%{+YYYY.MM.dd}"
  }
}
```

### **Создание панелей мониторинга**

#### **Kibana Dashboard для TI**
```json
{
  "title": "MISP Threat Intelligence Dashboard",
  "panels": [
    {
      "type": "metric",
      "params": {
        "metric": "misp.threat_level",
        "breakdown": "misp.threat_level"
      },
      "title": "Threat Level Distribution"
    },
    {
      "type": "table", 
      "params": {
        "columns": ["src_ip", "misp.threat_level", "misp.event_count"],
        "rows": 10
      },
      "title": "Top Threat IPs"
    }
  ]
}
```

### **Автоматическое реагирование**

#### **Python скрипт для автоматического блокирования**
```python
#!/usr/bin/env python3
from elasticsearch import Elasticsearch
from pymisp import ExpandedPyMISP
import requests

class AutoResponder:
    def __init__(self, es_host, misp_url, misp_key, firewall_api):
        self.es = Elasticsearch([es_host])
        self.misp = ExpandedPyMISP(misp_url, misp_key, ssl=False)
        self.firewall_api = firewall_api
        
    def monitor_alerts(self):
        """Мониторинг новых алертов с высоким уровнем угрозы"""
        query = {
            "query": {
                "bool": {
                    "must": [
                        {"term": {"misp.threat_level": "high"}},
                        {"range": {"@timestamp": {"gte": "now-5m"}}}
                    ]
                }
            }
        }
        
        response = self.es.search(
            index="enriched-logs-*",
            body=query
        )
        
        for hit in response['hits']['hits']:
            source = hit['_source']
            if source.get('src_ip'):
                self.block_ip(source['src_ip'])
                
    def block_ip(self, ip_address):
        """Блокировка IP в firewall"""
        block_payload = {
            "rule": {
                "name": f"MISP_BLOCK_{ip_address}",
                "source": ip_address,
                "action": "deny"
            }
        }
        
        response = requests.post(
            f"{self.firewall_api}/rules",
            json=block_payload,
            headers={'Authorization': 'Bearer FIREWALL_TOKEN'}
        )
        
        if response.status_code == 200:
            print(f"Successfully blocked IP: {ip_address}")
            
        # Добавление тега в MISP
        self.misp.tag(ip_address, "blocked-in-firewall")

# Запуск мониторинга
responder = AutoResponder(
    es_host='localhost:9200',
    misp_url='https://localhost', 
    misp_key='YOUR_API_KEY',
    firewall_api='https://firewall-api.local'
)

while True:
    responder.monitor_alerts()
    time.sleep(60)  # Проверка каждую минуту
```

### **Метрики эффективности**

#### **Отслеживание эффективности TI**
```python
# Скрипт для сбора метрик
def calculate_ti_metrics():
    metrics = {
        'total_iocs': 0,
        'high_confidence_iocs': 0,
        'iocs_used_in_detection': 0,
        'false_positives': 0,
        'mean_time_to_detect': 0
    }
    
    # Логика расчета метрик
    # ...
    
    return metrics
```
