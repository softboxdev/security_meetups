
### **Построение SIEM на Open-Source для российского рынка**

Это не просто установка ПО, а создание полноценного операционного центра безопасности (SOC). Рассмотрим архитектуру на основе проверенного стека **ELK (Elasticsearch, Logstash, Kibana) + Wazuh**.

---

### **1. Зачем это нужно? Соответствие законодательству РФ**

Прежде чем строить, понимаем, *зачем* и *что требует закон*:

*   **152-ФЗ "О персональных данных"**: Требует обеспечения конфиденциальности ПДн и регистрации всех операций с ними. **SIEM — центральный инструмент для этого.**
*   **Приказы ФСТЭК России (например, №17, №21)**: Требуют обнаружения вторжений, управления инцидентами, анализа уязвимостей и непрерывного мониторинга. Wazuh закрывает эти требования.
*   **Федеральный закон №187-ФЗ "О безопасности КИИ"**: Для объектов КИИ мониторинг и обнаружение атак являются обязательными.
*   **Приказ №239 Минцифры**: Требует проведения мониторинга безопасности информации.

**Вывод:** Open-source SIEM позволяет выполнить эти требования с минимальными затратами на лицензии, но с высокими затратами на экспертизу.

---

### **2. Выбор и обоснование стека технологий**

**Ядро: Wazuh + Elastic Stack (ELK)**

| Компонент | Роль в SIEM | Российская специфика |
| :--- | :--- | :--- |
| **Wazuh** | **Движок обнаружения угроз (HIDS, NIDS).** Собирает и коррелирует события с агентов, анализирует уязвимости, проверяет соответствие стандартам (CIS). | **Критически важен.** Фактически, это open-source аналог дорогих коммерческих EDR/NGFW. Позволяет выполнить многие требования ФСТЭК по обнаружению атак. |
| **Elasticsearch** | **Хранилище данных.** Масштабируемая NoSQL база для логов и событий безопасности. | Можно развернуть на российском железе в собственном ЦОДе. Проблем с санкциями на сам Elasticsearch нет. |
| **Logstash** | **"Трубопровод" данных.** Принимает, парсит, фильтрует, обогащает и перенаправляет логи из любых источников. | Позволяет адаптировать систему под любые российские источники (1С, Asterisk, DLP, российские СЗИ). |
| **Kibana** | **Визуализация и расследование.** Единый интерфейс для дашбордов, поиска по логам и анализа инцидентов. | **Внимание!** С 2021 года Elasticsearch и Kibana — проприетарные (Elastic License). **Альтернатива:** OpenSearch (форк от AWS, полностью open-source). |

**Архитектурная схема:**

```
[Источники логов] --> [Logstash / Wazuh Manager] --> [Elasticsearch] --> [Kibana/OpenSearch Dashboards]
     (Серверы, ПК,           (Центральный сервер          (Хранилище)         (Интерфейс для
      сетевые устройства)      для приема и обработки)                        аналитиков SOC)
```

---

### **3. Пошаговый план развертывания и настройки**

#### **Шаг 1: Развертывание инфраструктуры**

1.  **Серверы:** Разверните виртуальные машины (минимум 3 для отказоустойчивости) в вашем ЦОД. Используйте российские ОС (ALT Linux, ROSA) или проверенные Ubuntu/CentOS.
2.  **Установка стека:**
    *   Установите **Elasticsearch** (или **OpenSearch**) кластер для отказоустойчивости и производительности.
    *   Установите **Wazuh Manager** (отдельный сервер или виртуальная машина).
    *   Установите **Kibana** (или **OpenSearch Dashboards**) с плагином **Wazuh**.
3.  **Безопасность:** Изолируйте серверы SIEM в отдельном VLAN/VXLAN. Настройте строгие правила МЭ (Firewall). Все внутренние коммуникации между компонентами — по TLS с собственным CA.

#### **Шаг 2: Настройка сбора данных и интеграций**

Это самый важный этап. Источники должны покрывать всю ИТ-инфраструктуру.

1.  **Установка агентов Wazuh:**
    *   На все серверы (веб-приложения, БД, файловые хранилища).
    *   На рабочие станции администраторов и бухгалтерии (где обрабатываются ПДн).
    *   **Преимущество:** Агенты шифруют и аутентифицируются при передаче данных на Manager.

2.  **Настройка Logstash для российского стека:**
    *   **1С:** Парсинг логов файлов `*.lgp`, `*.lgf`. Направлять в SIEM события входа, изменения записей, особенно с полями ПДн.
    *   **Астериск/IP-телефония:** Сбор логов вызовов для мониторинга утечек по голосовым каналам.
    *   **Российские DLP, МЭ, СКАД:** Настройте отправку syslog от этих систем в Logstash. Они часто являются источником критически важных событий.
    *   **Маршрутизаторы (Cisco, MikroTik):** Отправка Netflow и syslog для обнаружения аномального сетевого трафика.

#### **Шаг 3: Создание правил корреляции и детектирования угроз**

"Мозг" SIEM. Без этого это просто сборник логов.

1.  **Используйте встроенные правила Wazuh:** Они покрывают 90% базовых атак (брутфорс, подозрительные процессы, изменение реестра).
2.  **Напишите кастомные правила под вашу инфраструктуру (Logstash фильтры или правила Wazuh):**
    *   **Правило №1:** "Если с одного IP >5 неудачных входов в 1С за 2 минуты -> **Критическое** предупреждение о брутфорсе".
    *   **Правило №2:** "Если пользователь бухгалтерии вошел в систему из Москвы, а через 10 минут — из Хабаровска -> **Высокий** риск компрометации учетной записи".
    *   **Правило №3 (для 152-ФЗ):** "Если запрос к API БД с ПДн возвращает >1000 записей -> **Средний** риск утечки. Отправить оповещение".
    *   **Правило №4:** "Обнаружено отключение агента Wazuh на сервере с ПДн -> **Критическое** предупреждение о возможной активности злоумышленника".

#### **Шаг 4: Визуализация и оповещение

1.  **Дашборды в Kibana/OpenSearch Dashboards:**
    *   Создайте дашборд "Безопасность ПДн" с графиками: "Топ пользователей, запрашивающих ПДн", "Геолокация входов в системы с ПДн".
    *   Дашборд "Инциденты ИБ" с общей статистикой по угрозам.
2.  **Настройка оповещений:**
    *   Интегрируйте с российскими системами коммуникации: **Telegram Bot API** (для срочных оповещений), **Email** (для официальных уведомлений).
    *   Настройте эскалацию: "Критическое" -> Telegram в 3 ночи, "Высокое" -> Email утром.

---

### **4. Ключевые рекомендации для российского рынка**

1.  **Суверенитет и локализация:**
    *   Развертывайте на своем оборудовании или у российского облачного провайдера.
    *   Рассмотрите российские форки или аналоги (OpenSearch вместо Elasticsearch).
    *   Все данные шифруйтесь на стороне заказчика.

2.  **Фокус на 152-ФЗ:**
    *   Обязательно настройте правила, отслеживающие любые операции с персональными данными (чтение, изменение, удаление, массовый экспорт).
    *   Ведите журналы не менее 6 месяцев (или иного срока, указанного в вашем частном случае для 152-ФЗ).

3.  **Процессы, а не технологии:**
    *   **Создайте регламент работы SOC:** Кто и как реагирует на оповещения? Кто и когда закрывает инцидент?
    *   **Ведите базу инцидентов:** Каждый сработавший алерт должен быть зафиксирован как инцидент с описанием шагов по устранению. Это прямое требование ФСТЭК.
    *   **Проводите регулярные тренировки** для аналитиков.

### **Итог: Плюсы и Минусы**

| Плюсы (За) | Минусы (Против) |
| :--- | :--- |
| **Полный контроль** над данными и логикой. | **Высокие затраты на экспертизу** (нужен сильный Linux-инженер и аналитик ИБ). |
| **Соответствие 152-ФЗ и ФСТЭК** без дорогих лицензий. | **Требует постоянной донастройки** и техобслуживания (это "живой" организм). |
| **Гибкость** для интеграции любого российского ПО. | **Ограниченная техподдержка.** Вы решаете проблемы сами или через комьюнити. |
| **Масштабируемость** под нужды бизнеса. | **Время развертывания** значительно выше, чем у "коробочной" SIEM. |

**Заключение:** Построение open-source SIEM — это стратегическое решение для компании, которая готова инвестировать в собственную экспертизу и хочет получить максимальный контроль и соответствие российскому законодательству при минимальных капитальных затратах.


---

### **Подробный пошаговый план развертывания и настройки SIEM**

**Предварительные условия:**
*   **Аппаратные ресурсы:** Минимум 3 ВМ/сервера (8+ vCPU, 16+ GB RAM, 500+ GB SSD) для кластера.
*   **ПО:** Ubuntu Server 22.04 LTS / CentOS 7+ (или российский дистрибутив, например, ALT Linux).
*   **Доступ:** Права `root` или пользователя с `sudo`.
*   **Сеть:** Статические IP-адреса, настроенный `hostname`, открытые порты между компонентами.

---

### **Архитектура развертывания**

```
[Агенты Wazuh] --> [Wazuh Manager] --> [OpenSearch] <--> [OpenSearch Dashboards]
                         |
                         +--> [Filebeat] --> [OpenSearch]
```

---

### **ШАГ 1: Подготовка инфраструктуры и базовое hardening ОС**

**Цель:** Создать безопасную и отказоустойчивую основу.

1.  **Развертывание ВМ:**
    *   `siem-node1` (16 GB RAM, 4 vCPU, 200 GB) - **Wazuh Manager + OpenSearch**
    *   `siem-node2` (16 GB RAM, 4 vCPU, 200 GB) - **OpenSearch**
    *   `siem-node3` (8 GB RAM, 2 vCPU, 100 GB) - **OpenSearch Dashboards**

2.  **Базовая настройка на всех узлах:**
    ```bash
    # Обновление системы
    sudo apt update && sudo apt upgrade -y  # Для Debian/Ubuntu
    # sudo yum update -y # Для CentOS/RHEL

    # Установка базовых утилит
    sudo apt install -y curl wget vim net-tools

    # Настройка hostname и hosts (на каждой ноде)
    sudo hostnamectl set-hostname siem-node1 # и т.д.
    sudo vim /etc/hosts
    # Добавить строки:
    # 192.168.1.10 siem-node1
    # 192.168.1.11 siem-node2
    # 192.168.1.12 siem-node3
    ```

3. **Настройка базовой безопасности (Hardening):**
    ```bash
    # Настройка iptables/ufw (пример для ufw)
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow from 192.168.1.0/24 to any port 22
    sudo ufw enable

    # Отключение SELinux (если используется) или настройка в permissive mode
    sudo setenforce 0
    sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
    ```

---

### **ШАГ 2: Установка и настройка OpenSearch (хранилище)**

**Цель:** Развернуть отказоустойчивое хранилище для логов.

1. **Установка Java (на всех узлах OpenSearch):**
    ```bash
    sudo apt install -y openjdk-17-jdk
    java -version
    ```

2. **Настройка системных параметров для OpenSearch:**
    ```bash
    # Увеличим лимиты памяти
    sudo vim /etc/sysctl.conf
    # Добавить:
    # vm.max_map_count=262144
    
    sudo sysctl -p

    # Настроим limits
    sudo vim /etc/security/limits.conf
    # Добавить:
    # opensearch soft memlock unlimited
    # opensearch hard memlock unlimited
    ```

3. **Установка OpenSearch (на siem-node1 и siem-node2):**
    ```bash
    # Скачивание пакета
    wget https://artifacts.opensearch.org/releases/bundle/opensearch/2.11.0/opensearch-2.11.0-linux-x64.tar.gz
    tar -xzf opensearch-2.11.0-linux-x64.tar.gz
    sudo mv opensearch-2.11.0 /usr/share/opensearch

    # Создание пользователя и прав
    sudo useradd --system --shell /bin/bash --home-dir /usr/share/opensearch opensearch
    sudo chown -R opensearch:opensearch /usr/share/opensearch
    ```

4. **Конфигурация OpenSearch (`/usr/share/opensearch/config/opensearch.yml`):**
    ```yaml
    # На siem-node1
    cluster.name: siem-cluster
    node.name: siem-node1
    node.roles: [cluster_manager, data]
    network.host: 0.0.0.0
    discovery.seed_hosts: ["siem-node1", "siem-node2"]
    cluster.initial_cluster_manager_nodes: ["siem-node1"]

    # На siem-node2  
    cluster.name: siem-cluster
    node.name: siem-node2
    node.roles: [data]
    network.host: 0.0.0.0
    discovery.seed_hosts: ["siem-node1", "siem-node2"]
    cluster.initial_cluster_manager_nodes: ["siem-node1"]
    ```

5. **Запуск OpenSearch:**
    ```bash
    sudo -u opensearch /usr/share/opensearch/bin/opensearch
    # Проверить работу: curl -XGET https://siem-node1:9200 -u 'admin:admin' --insecure
    ```

---

### **ШАГ 3: Установка и настройка Wazuh Manager**

**Цель:** Развернуть ядро системы обнаружения угроз.

1. **Установка Wazuh Manager (на siem-node1):**
    ```bash
    # Добавление репозитория
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
    sudo apt update
    sudo apt install -y wazuh-manager
    sudo systemctl daemon-reload
    sudo systemctl enable wazuh-manager
    sudo systemctl start wazuh-manager
    ```

2. **Проверка статуса Wazuh Manager:**
    ```bash
    sudo systemctl status wazuh-manager
    # Проверить логи: tail -f /var/ossec/logs/ossec.log
    ```

---

### **ШАГ 4: Установка OpenSearch Dashboards (интерфейс)**

**Цель:** Развернуть веб-интерфейс для визуализации и анализа.

1. **Установка на siem-node3:**
    ```bash
    wget https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/2.11.0/opensearch-dashboards-2.11.0-linux-x64.tar.gz
    tar -xzf opensearch-dashboards-2.11.0-linux-x64.tar.gz
    sudo mv opensearch-dashboards-2.11.0 /usr/share/opensearch-dashboards
    sudo useradd --system --shell /bin/bash --home-dir /usr/share/opensearch-dashboards opensearch-dashboards
    sudo chown -R opensearch-dashboards:opensearch-dashboards /usr/share/opensearch-dashboards
    ```

2. **Конфигурация (`/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml`):**
    ```yaml
    server.port: 5601
    server.host: "0.0.0.0"
    opensearch.hosts: ["https://siem-node1:9200"]
    opensearch.ssl.verificationMode: none
    opensearch.username: "admin"
    opensearch.password: "admin"
    opensearch.requestHeadersWhitelist: ["securitytenant", "Authorization"]
    opensearch_security.multitenancy.enabled: true
    opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
    opensearch_security.readonly_mode.roles: ["kibana_read_only"]
    # Использование русского языка
    i18n.locale: "ru"
    ```

3. **Запуск OpenSearch Dashboards:**
    ```bash
    sudo -u opensearch-dashboards /usr/share/opensearch-dashboards/bin/opensearch-dashboards
    ```

---

### **ШАГ 5: Интеграция Wazuh с OpenSearch**

**Цель:** Настроить передачу данных от Wazuh в OpenSearch.

1. **Установка Filebeat (на siem-node1):**
    ```bash
    curl -s https://packages.wazuh.com/4.x/apt/pool/main/f/filebeat/filebeat-7.10.2-amd64.deb -o filebeat.deb
    sudo dpkg -i filebeat.deb
    ```

2. **Конфигурация Filebeat (`/etc/filebeat/filebeat.yml`):**
    ```yaml
    output.opensearch:
      hosts: ["siem-node1:9200"]
      protocol: "https"
      username: "admin"
      password: "admin"
      ssl:
        verification_mode: "none"

    filebeat.modules:
      - module: wazuh
        alerts:
          enabled: true
        archives:
          enabled: false

    setup.template.enabled: true
    setup.template.name: "wazuh"
    setup.template.pattern: "wazuh-alerts-4.x-*"
    setup.ilm.enabled: false
    ```

3. **Запуск и настройка Filebeat:**
    ```bash
    sudo filebeat modules enable wazuh
    sudo filebeat setup --index-management
    sudo systemctl daemon-reload
    sudo systemctl enable filebeat
    sudo systemctl start filebeat
    ```

---

### **ШАГ 6: Установка Wazuh плагина для OpenSearch Dashboards**

**Цель:** Добавить специализированный интерфейс для Wazuh.

1. **Установка плагина:**
    ```bash
    # На siem-node3
    sudo -u opensearch-dashboards /usr/share/opensearch-dashboards/bin/opensearch-dashboards-plugin install https://packages.wazuh.com/4.x/ui/dashboard/wazuh-4.7.2_2.11.0.zip
    ```

2. **Перезапуск OpenSearch Dashboards:**
    ```bash
    sudo systemctl restart opensearch-dashboards
    ```

---

### **ШАГ 7: Установка и регистрация Wazuh агентов**

**Цель:** Подключить конечные узлы к системе мониторинга.

1. **Установка агента на Linux-сервер:**
    ```bash
    # Добавление репозитория (аналогично шагу 3)
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
    sudo apt update
    sudo apt install -y wazuh-agent
    ```

2. **Регистрация агента:**
    ```bash
    sudo /var/ossec/bin/agent-auth -m siem-node1 -A "web-server-01"
    ```

3. **Настройка агента (`/var/ossec/etc/ossec.conf`):**
    ```xml
    <ossec_config>
      <client>
        <server>
          <address>siem-node1</address>
          <port>1514</port>
          <protocol>tcp</protocol>
        </server>
        <config_profile>ubuntu, ubuntu20</config_profile>
        <notify_time>60</notify_time>
        <time-reconnect>300</time-reconnect>
      </client>
    </ossec_config>
    ```

4. **Запуск агента:**
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable wazuh-agent
    sudo systemctl start wazuh-agent
    ```

---

### **ШАГ 8: Настройка системных служб и автоматического запуска**

**Цель:** Обеспечить автоматический запуск всех компонентов при перезагрузке.

1. **Создание systemd служб для OpenSearch и OpenSearch Dashboards:**
    
    **Для OpenSearch (`/etc/systemd/system/opensearch.service`):**
    ```ini
    [Unit]
    Description=OpenSearch
    Documentation=https://opensearch.org/
    After=network.target

    [Service]
    User=opensearch
    Group=opensearch
    WorkingDirectory=/usr/share/opensearch
    ExecStart=/usr/share/opensearch/bin/opensearch
    Restart=always
    RestartSec=3

    [Install]
    WantedBy=multi-user.target
    ```

    **Для OpenSearch Dashboards (`/etc/systemd/system/opensearch-dashboards.service`):**
    ```ini
    [Unit]
    Description=OpenSearch Dashboards
    Documentation=https://opensearch.org/
    After=network.target

    [Service]
    User=opensearch-dashboards
    Group=opensearch-dashboards
    WorkingDirectory=/usr/share/opensearch-dashboards
    ExecStart=/usr/share/opensearch-dashboards/bin/opensearch-dashboards
    Restart=always
    RestartSec=3

    [Install]
    WantedBy=multi-user.target
    ```

2. **Активация служб:**
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable opensearch opensearch-dashboards
    sudo systemctl start opensearch opensearch-dashboards
    ```

---

### **ШАГ 9: Базовые проверки работоспособности**

**Цель:** Убедиться, что все компоненты работают корректно.

1. **Проверка статуса служб:**
    ```bash
    sudo systemctl status opensearch
    sudo systemctl status opensearch-dashboards  
    sudo systemctl status wazuh-manager
    sudo systemctl status filebeat
    ```

2. **Проверка через веб-интерфейс:**
    * Откройте в браузере: `http://siem-node3:5601`
    * Логин: `admin` / `admin`
    * В меню должен появиться пункт "Wazuh"

3. **Проверка индексов в OpenSearch:**
    ```bash
    curl -XGET 'https://siem-node1:9200/_cat/indices?v' -u 'admin:admin' --insecure
    # Должны отобразиться индексы wazuh-alerts-*
    ```

4. **Проверка подключенных агентов:**
    ```bash
    sudo /var/ossec/bin/agent_control -l
    ```

---

### **Дальнейшие шаги после базовой установки**

1. **Настройка SSL/TLS сертификатов** для всех компонентов
2. **Настройка резервного копирования** индексов OpenSearch
3. **Создание кастомных правил обнаружения** под вашу инфраструктуру
4. **Настройка алертинга** (Email, Telegram, Slack)
5. **Интеграция дополнительных источников логов** (1С, сетевые устройства)
6. **Создание дашбордов** для мониторинга compliance с 152-ФЗ

Этот план дает полноценную рабочую SIEM систему, которую можно развивать и адаптировать под конкретные требования вашей организации и российского законодательства.


---

---

## **Детальная настройка взаимодействия компонентов SIEM**

### **1. Сетевая архитектура и план адресации**

**Предполагаемая сеть:** `192.168.1.0/24`

| Сервер | IP адрес | Hostname | Роли | Критические порты |
|--------|-----------|-----------|-------|-------------------|
| **siem-node1** | `192.168.1.10` | `siem-node1.local` | Wazuh Manager + OpenSearch Master | 9200, 9300, 1514, 1515, 55000 |
| **siem-node2** | `192.168.1.11` | `siem-node2.local` | OpenSearch Data Node | 9200, 9300 |
| **siem-node3** | `192.168.1.12` | `siem-node3.local` | OpenSearch Dashboards | 5601 |
| **Агенты** | `192.168.1.100-200` | - | Wazuh Agents | 1514 (outbound) |

---

### **2. Подготовка DNS / hosts файлов**

**На всех серверах правим `/etc/hosts`:**
```bash
sudo vim /etc/hosts

# Добавляем строки:
192.168.1.10 siem-node1 siem-node1.local
192.168.1.11 siem-node2 siem-node2.local  
192.168.1.12 siem-node3 siem-node3.local
```

**Проверяем разрешение имен:**
```bash
ping siem-node1.local
ping siem-node2.local
ping siem-node3.local
```

---

### **3. Настройка OpenSearch кластера**

#### **3.1. Конфигурация siem-node1 (`/usr/share/opensearch/config/opensearch.yml`)**

```yaml
# Basic cluster configuration
cluster.name: siem-cluster
node.name: siem-node1
node.roles: [cluster_manager, data, ingest]

# Network settings
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# Discovery and cluster formation
discovery.seed_hosts: ["siem-node1:9300", "siem-node2:9300"]
cluster.initial_cluster_manager_nodes: ["siem-node1"]

# Security configuration (basic authentication)
plugins.security.ssl.transport.pemcert_filepath: node1.pem
plugins.security.ssl.transport.pemkey_filepath: node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.http.enabled: false  # Отключаем для начальной настройки

# Performance optimization
bootstrap.memory_lock: true
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch

# Russian market compliance - индексы по месяцам
action.auto_create_index: ".opendistro-alerting-config,.opendistro-alerting-alert*,-*"
```

#### **3.2. Конфигурация siem-node2 (`/usr/share/opensearch/config/opensearch.yml`)**

```yaml
cluster.name: siem-cluster
node.name: siem-node2
node.roles: [data, ingest]  # Только data node

# Network settings
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# Discovery
discovery.seed_hosts: ["siem-node1:9300", "siem-node2:9300"]
cluster.initial_cluster_manager_nodes: ["siem-node1"]

# Security (аналогично node1)
plugins.security.ssl.http.enabled: false

# Performance
bootstrap.memory_lock: true
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch
```

#### **3.3. Настройка системных лимитов**

**На всех узлах OpenSearch создаем `/etc/systemd/system/opensearch.service`:**
```ini
[Unit]
Description=OpenSearch
Documentation=https://opensearch.org/
After=network.target

[Service]
Type=simple
User=opensearch
Group=opensearch
WorkingDirectory=/usr/share/opensearch
ExecStart=/usr/share/opensearch/bin/opensearch
Restart=always
RestartSec=3
LimitMEMLOCK=infinity
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

**Настройка ulimits (`/etc/security/limits.conf`):**
```bash
opensearch soft memlock unlimited
opensearch hard memlock unlimited
opensearch soft nofile 65536
opensearch hard nofile 65536
opensearch soft nproc 4096
opensearch hard nproc 4096
```

---

### **4. Настройка Wazuh Manager на siem-node1**

#### **4.1. Основная конфигурация (`/var/ossec/etc/ossec.conf`)**

```xml
<ossec_config>
  <!-- Global Settings -->
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>no</logall>
    <logall_json>no</logall_json>
  </global>

  <!-- Integration with OpenSearch -->
  <integration>
    <name>opensearch</name>
    <hook_url>http://localhost:9200</hook_url>
    <level>3</level>
    <alert_format>json</alert_format>
  </integration>

  <!-- Email notifications (Russian SMTP) -->
  <global>
    <email_notification>yes</email_notification>
    <smtp_server>smtp.yandex.ru</smtp_server>
    <smtp_port>587</smtp_port>
    <email_from>siem@yourcompany.ru</email_from>
    <smtp_username>siem@yourcompany.ru</smtp_username>
    <smtp_password>your_password</smtp_password>
    <email_to>security-team@yourcompany.ru</email_to>
  </global>

  <!-- Remote agent communication -->
  <remote>
    <connection>secure</connection>
    <port>1514</port>
    <protocol>tcp</protocol>
    <allowed-ips>192.168.1.0/24</allowed-ips>
  </remote>

  <!-- Local file monitoring -->
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>

  <!-- Active Response -->
  <command>
    <name>firewall-drop</name>
    <executable>firewall-drop.sh</executable>
    <expect>srcip</expect>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <level>10</level>
    <timeout>600</timeout>
  </active-response>
</ossec_config>
```

#### **4.2. Настройка Filebeat для интеграции с OpenSearch**

**Конфигурация (`/etc/filebeat/filebeat.yml`):**

```yaml
# Filebeat configuration for Wazuh
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
      var.paths: ["/var/ossec/logs/alerts/alerts.json"]
    archives:
      enabled: false

# OpenSearch output configuration
output.opensearch:
  hosts: ["192.168.1.10:9200", "192.168.1.11:9200"]
  protocol: "https"
  username: "admin"
  password: "admin"
  ssl:
    certificate_authorities: ["/etc/filebeat/root-ca.pem"]
    certificate: "/etc/filebeat/filebeat.pem"
    key: "/etc/filebeat/filebeat-key.pem"
    verification_mode: "full"

# Setup
setup.template.enabled: true
setup.template.name: "wazuh"
setup.template.pattern: "wazuh-alerts-4.x-*"
setup.ilm.enabled: false

# Logging
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644
```

---

### **5. Настройка OpenSearch Dashboards на siem-node3**

#### **5.1. Конфигурация (`/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml`)**

```yaml
# Server settings
server.port: 5601
server.host: "0.0.0.0"
server.name: "siem-dashboard"
server.ssl.enabled: false

# OpenSearch connection
opensearch.hosts: ["http://192.168.1.10:9200", "http://192.168.1.11:9200"]
opensearch.ssl.verificationMode: none
opensearch.username: "admin"
opensearch.password: "admin"
opensearch.requestHeadersWhitelist: ["securitytenant", "Authorization"]

# Security plugin
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
opensearch_security.readonly_mode.roles: ["kibana_read_only"]

# Internationalization
i18n.locale: "ru"

# Wazuh plugin configuration
wazuh.security.enabled: true
wazuh.security.xpack.rbac.enabled: true

# Performance
opensearch.healthCheck.delay: 120000
opensearch.initialRetryDelay: 10000
opensearch.maxRetryDelay: 120000

# Logging
logging.verbose: true
logging.dest: /var/log/opensearch-dashboards.log
```

#### **5.2. Systemd служба для OpenSearch Dashboards**

**Создаем `/etc/systemd/system/opensearch-dashboards.service`:**
```ini
[Unit]
Description=OpenSearch Dashboards
Documentation=https://opensearch.org/
After=network.target

[Service]
Type=simple
User=opensearch-dashboards
Group=opensearch-dashboards
WorkingDirectory=/usr/share/opensearch-dashboards
Environment=NODE_ENV=production
Environment=OPENSEARCH_HOSTS=http://192.168.1.10:9200
ExecStart=/usr/share/opensearch-dashboards/bin/opensearch-dashboards
Restart=always
RestartSec=3
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

---

### **6. Настройка межсерверного взаимодействия**

#### **6.1. Настройка firewall (UFW) на всех серверах**

**На siem-node1:**
```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

# OpenSearch ports
sudo ufw allow from 192.168.1.0/24 to any port 9200
sudo ufw allow from 192.168.1.0/24 to any port 9300

# Wazuh Manager ports
sudo ufw allow from 192.168.1.0/24 to any port 1514
sudo ufw allow from 192.168.1.0/24 to any port 1515
sudo ufw allow from 192.168.1.0/24 to any port 55000

# SSH access
sudo ufw allow from 192.168.1.0/24 to any port 22
```

**На siem-node2:**
```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

# OpenSearch ports
sudo ufw allow from 192.168.1.0/24 to any port 9200
sudo ufw allow from 192.168.1.0/24 to any port 9300
sudo ufw allow from 192.168.1.12 to any port 5601

# SSH access
sudo ufw allow from 192.168.1.0/24 to any port 22
```

**На siem-node3:**
```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

# OpenSearch Dashboards port
sudo ufw allow from 192.168.1.0/24 to any port 5601

# SSH access
sudo ufw allow from 192.168.1.0/24 to any port 22
```

#### **6.2. Проверка сетевой связности**

**Скрипт проверки (`check_connectivity.sh`):**
```bash
#!/bin/bash

echo "=== Checking SIEM Cluster Connectivity ==="

# Check from siem-node1
echo "1. Checking OpenSearch cluster..."
curl -X GET "http://192.168.1.10:9200/_cluster/health?pretty"
curl -X GET "http://192.168.1.11:9200/_cluster/health?pretty"

echo "2. Checking Wazuh Manager..."
telnet 192.168.1.10 1514

echo "3. Checking OpenSearch Dashboards..."
curl -X GET "http://192.168.1.12:5601/api/status"

echo "4. Checking node connectivity..."
ping -c 2 siem-node2.local
ping -c 2 siem-node3.local

echo "=== Connectivity Check Complete ==="
```

---

### **7. Настройка резервного копирования и мониторинга**

#### **7.1. Резервное копирование конфигураций**

**Скрипт бэкапа (`/opt/scripts/backup_siem_config.sh`):**
```bash
#!/bin/bash
BACKUP_DIR="/backup/siem-config"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR/$DATE

# Backup Wazuh configuration
cp -r /var/ossec/etc $BACKUP_DIR/$DATE/wazuh-etc
cp -r /var/ossec/ruleset $BACKUP_DIR/$DATE/wazuh-ruleset

# Backup OpenSearch configuration
cp -r /usr/share/opensearch/config $BACKUP_DIR/$DATE/opensearch-config

# Backup OpenSearch Dashboards configuration  
cp -r /usr/share/opensearch-dashboards/config $BACKUP_DIR/$DATE/dashboards-config

# Create archive
tar -czf $BACKUP_DIR/siem_backup_$DATE.tar.gz $BACKUP_DIR/$DATE

# Cleanup old backups (keep 7 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_DIR/siem_backup_$DATE.tar.gz"
```

#### **7.2. Мониторинг состояния кластера**

**Скрипт мониторинга (`/opt/scripts/monitor_siem_cluster.sh`):**
```bash
#!/bin/bash

# Check OpenSearch cluster health
CLUSTER_HEALTH=$(curl -s -X GET "http://192.168.1.10:9200/_cluster/health" | jq -r '.status')
if [ "$CLUSTER_HEALTH" != "green" ]; then
    echo "ALERT: OpenSearch cluster status is $CLUSTER_HEALTH" | mail -s "SIEM Alert" security-team@yourcompany.ru
fi

# Check Wazuh Manager
if ! systemctl is-active --quiet wazuh-manager; then
    echo "ALERT: Wazuh Manager is down" | mail -s "SIEM Alert" security-team@yourcompany.ru
fi

# Check OpenSearch Dashboards
if ! curl -s http://192.168.1.12:5601 > /dev/null; then
    echo "ALERT: OpenSearch Dashboards is down" | mail -s "SIEM Alert" security-team@yourcompany.ru
fi

# Check disk space
DISK_USAGE=$(df / | awk 'END{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "ALERT: Disk usage is $DISK_USAGE%" | mail -s "SIEM Alert" security-team@yourcompany.ru
fi
```

---

### **8. Порядок запуска системы**

#### **8.1. Последовательность запуска сервисов**

```bash
# На siem-node1 и siem-node2:
sudo systemctl start opensearch
sudo systemctl enable opensearch

# На siem-node1:
sudo systemctl start wazuh-manager
sudo systemctl enable wazuh-manager
sudo systemctl start filebeat
sudo systemctl enable filebeat

# На siem-node3:
sudo systemctl start opensearch-dashboards
sudo systemctl enable opensearch-dashboards
```

#### **8.2. Проверка работоспособности**

**Команды для проверки:**
```bash
# Проверить кластер OpenSearch
curl -XGET "http://192.168.1.10:9200/_cat/nodes?v"
curl -XGET "http://192.168.1.10:9200/_cat/indices/wazuh*?v"

# Проверить Wazuh Manager
sudo systemctl status wazuh-manager
/var/ossec/bin/agent_control -l

# Проверить OpenSearch Dashboards
curl -XGET "http://192.168.1.12:5601/api/status"
```

---

### **9. Источники конфигураций и документации**

1. **Официальная документация:**
   - OpenSearch: https://opensearch.org/docs/latest/
   - Wazuh: https://documentation.wazuh.com/current/index.html

2. **Порты по умолчанию:**
   - OpenSearch: 9200 (HTTP), 9300 (Transport)
   - Wazuh Manager: 1514 (Agents), 1515 (Auth), 55000 (Cluster)
   - OpenSearch Dashboards: 5601 (Web UI)

3. **Конфигурационные файлы:**
   - OpenSearch: `/usr/share/opensearch/config/opensearch.yml`
   - Wazuh: `/var/ossec/etc/ossec.conf`
   - OpenSearch Dashboards: `/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml`


Подборка справочных материалов, документации и практических ресурсов, которые помогут вам в развертывании и настройке SIEM-системы на основе Wazuh и OpenSearch.

### 📚 Официальная документация и практические руководства

Это основа для работы с рассмотренными технологиями. Официальная документация содержит самые актуальные и достоверные сведения.

| Ресурс | Описание |
| :--- | :--- |
| [Документация Wazuh: Интеграция с Elastic Stack](https://documentation.wazuh.com/current/integrations-guide/elastic-stack/index.html)  | Подробное руководство по интеграции Wazuh с Elastic Stack (включая OpenSearch) с использованием Logstash. Содержит инструкции по установке, настройке конвейеров и шаблонов индексов. |
| [Документация Wazuh: Пользовательские правила](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)  | Исчерпывающее руководство по созданию собственных правил для Wazuh. Включает примеры, объяснение синтаксиса и важные предупреждения о совместимости. |
| [Синтаксис правил Wazuh](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)  | Справочник по всем XML-тегам и параметрам, используемым для создания правил. Незаменим при написании сложных условий. |
| [Блог Wazuh: Расширение возможностей с помощью Elastic Stack](https://wazuh.com/blog/detection-with-elastic-stack-integration/)  | Практический пример использования связки Wazuh и Elastic Stack для обеспечения соответствия требованиям PCI DSS. |

### 🛠️ Ключевые аспекты настройки и практического применения

Для эффективного использования этих материалов стоит сосредоточиться на нескольких ключевых областях.

- **Создание пользовательских правил обнаружения угроз**
    - **Используйте правильный диапазон идентификаторов:** Для кастомных правил применяйте ID от `100000` до `120000`, чтобы избежать конфликтов с системными правилами .
    - **Структурируйте правила:** Помещайте правила в группы (`<group name="...">`) для логической организации и удобной фильтрации в дашбордах .
    - **Модификация существующих правил:** Чтобы изменить встроенное правило, скопируйте его в файл `/var/ossec/etc/rules/local_rules.xml` и добавьте атрибут `overwrite="yes"`. Это гарантирует, что ваши изменения не будут потеряны при обновлении системы .
    - **Пример практического правила:** Для пометки потенциальных ложных срабатываний от Suricata можно создать дочернее правило, которое добавляет информативный тег .
    ```xml
    <group name="ids,suricata,custom,possible_false_positive,">
      <rule id="100500" level="3">
        <if_sid>86601</if_sid>
        <field name="alert.signature">GPL ICMP_INFO PING</field>
        <description>Suricata: Alert - $(alert.signature)</description>
        <info type="text">Possible False Positive</info>
        <options>no_full_log</options>
      </rule>
    </group>
    ```

- **Настройка конвейеров данных и интеграций**
    - **Роль Logstash:** Logstash обеспечивает гибкость для сложной обработки данных перед их отправкой в хранилище (обогащение, фильтрация, маршрутизация в разные индексы) .
    - **Безопасное хранение секретов:** Используйте встроенное хранилище секретов Logstash (keystore) для безопасного управления учетными данными к базам данных, а не хранения их в plaintext-файлах .
    - **Настройка шаблонов индексов:** Заранее настраивайте маппинги для индексов OpenSearch/Elasticsearch. Шаблон от Wazuh автоматически увеличивает лимит на количество полей до 10000, что необходимо для корректного отображения всех данных .

### 💡 Дополнительные ресурсы для углубленного изучения

- **Пример базового развертывания:** Репозиторий [`mriazx/wazuh-setup`](https://github.com/mriazx/wazuh-setup) на GitHub может служить как отправная точка для понимания общей архитектуры развертывания .
- **Сообщество и поддержка:** Для решения специфических проблем используйте [Группы рассылки Wazuh](https://groups.google.com/g/wazuh). Это место, где можно задать вопрос и поучиться на реальных кейсах других пользователей .

