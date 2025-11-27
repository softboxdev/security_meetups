
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

## **Архитектурная схема взаимодействия компонентов**

```
[Агенты Wazuh] 
       ↓ (TCP 1514)
[Wazuh Manager] → [Filebeat] → (HTTPS 9200) → [OpenSearch Cluster]
       ↓ (TCP 1515 - для внешних логов)
[Logstash] → [OpenSearch]
                         ↗
[OpenSearch Dashboards] ← (HTTPS 9200)
```

---

## **ЧАСТЬ 1: Сетевые спецификации и порты**

### **1.1. Таблица сетевых настроек**

| Компонент | Сервер | IP-адрес | Интерфейс | VLAN |
|-----------|---------|----------|-----------|------|
| OpenSearch Node 1 | siem-node1 | 192.168.10.10 | ens192 | 10 |
| OpenSearch Node 2 | siem-node2 | 192.168.10.11 | ens192 | 10 |
| Wazuh Manager | siem-node1 | 192.168.10.10 | ens192 | 10 |
| OpenSearch Dashboards | siem-node3 | 192.168.10.12 | ens192 | 10 |
| Агенты | Все хосты | 192.168.0.0/16 | * | * |

### **1.2. Таблица портов и протоколов**

| Порт | Протокол | Назначение | Источник | Назначение |
|------|----------|------------|----------|------------|
| **9200** | HTTPS | OpenSearch API | Все компоненты | OpenSearch Nodes |
| **9300** | TCP | OpenSearch Transport | siem-node1 ↔ siem-node2 | OpenSearch Nodes |
| **5601** | HTTPS | Web-интерфейс | Пользователи | siem-node3 |
| **1514** | TCP | Агенты → Manager | Агенты | siem-node1 |
| **1515** | TCP/UDP | Syslog → Manager | Сетевые устройства | siem-node1 |
| **55000** | TCP | Filebeat → OpenSearch | siem-node1 | OpenSearch Nodes |

---

## **ЧАСТЬ 2: Детальная настройка каждого компонента**

### **2.1. OpenSearch Cluster Configuration**

**На siem-node1 (192.168.10.10) - `/etc/opensearch/opensearch.yml`:**
```yaml
# Basic configuration
cluster.name: siem-cluster
node.name: siem-node1
node.roles: [cluster_manager, data, ingest]

# Network
network.host: 192.168.10.10
http.port: 9200
transport.port: 9300

# Discovery and cluster formation
discovery.seed_hosts: ["192.168.10.10:9300", "192.168.10.11:9300"]
cluster.initial_cluster_manager_nodes: ["siem-node1"]

# Security (базовые настройки)
plugins.security.ssl.transport.pemcert_filepath: node1.pem
plugins.security.ssl.transport.pemkey_filepath: node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: node1.pem
plugins.security.ssl.http.pemkey_filepath: node1-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: root-ca.pem

# Russian language support
i18n.locale: ru
```

**На siem-node2 (192.168.10.11) - `/etc/opensearch/opensearch.yml`:**
```yaml
cluster.name: siem-cluster
node.name: siem-node2
node.roles: [data, ingest]

network.host: 192.168.10.11
http.port: 9200
transport.port: 9300

discovery.seed_hosts: ["192.168.10.10:9300", "192.168.10.11:9300"]
cluster.initial_cluster_manager_nodes: ["siem-node1"]

# Security settings (аналогично node1)
plugins.security.ssl.transport.pemcert_filepath: node2.pem
# ... остальные SSL настройки
```

### **2.2. Wazuh Manager Configuration**

**На siem-node1 - `/var/ossec/etc/ossec.conf`:**
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
    <hook_url>https://192.168.10.10:9200</hook_url>
    <api_key>your-opensearch-api-key</api_key>
    <level>3</level>
    <alert_format>json</alert_format>
  </integration>

  <!-- Authentication Settings -->
  <auth>
    <disabled>no</disabled>
    <port>1515</port>
    <use_source_ip>no</use_source_ip>
    <force_insert>yes</force_insert>
    <force_time>0</force_time>
    <purge>yes</purge>
    <use_password>no</use_password>
    <limit_maxagents>5000</limit_maxagents>
    <ciphers>HIGH:!ADH:!EXP:!MD5:!RC4:!3DES:!CAMELLIA:@STRENGTH</ciphers>
    <!-- SSL Settings -->
    <ssl_agent_ca>/var/ossec/etc/rootCA.pem</ssl_agent_ca>
    <ssl_verify_host>no</ssl_verify_host>
  </auth>

  <!-- Remote Agent Communication -->
  <remote>
    <connection>secure</connection>
    <port>1514</port>
    <protocol>tcp</protocol>
    <queue_size>131072</queue_size>
  </remote>

  <!-- Logging -->
  <logging>
    <log_format>json</log_format>
  </logging>

  <!-- Rules Configuration -->
  <ruleset>
    <rule_dir>ruleset/rules</rule_dir>
    <rule_dir>ruleset/sca</rule_dir>
    <rule_dir>etc/rules</rule_dir>  <!-- Кастомные правила -->
    <list>etc/lists/audit-keys</list>
    <email_alert_level>12</email_alert_level>
  </ruleset>

  <!-- Vulnerability Detector -->
  <vulnerability-detector>
    <enabled>yes</enabled>
    <interval>5m</interval>
    <ignore_time>6h</ignore_time>
    <run_on_start>yes</run_on_start>
    
    <!-- Ubuntu OS -->
    <provider name="canonical">
      <enabled>yes</enabled>
      <os>trusty</os>
      <os>xenial</os>
      <os>bionic</os>
      <os>focal</os>
      <os>jammy</os>
      <update_interval>1h</update_interval>
    </provider>

    <!-- RedHat OS -->
    <provider name="redhat">
      <enabled>yes</enabled>
      <os>5</os>
      <os>6</os>
      <os>7</os>
      <os>8</os>
      <os>9</os>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Windows OS -->
    <provider name="msu">
      <enabled>yes</enabled>
      <update_interval>1h</update_interval>
    </provider>
  </vulnerability-detector>

  <!-- Syslog Configuration (для сетевых устройств) -->
  <localfile>
    <location>/var/log/syslog</location>
    <log_format>syslog</log_format>
  </localfile>

  <!-- Active Response -->
  <active-response>
    <disabled>no</disabled>
    <ca_verification>no</ca_verification>
  </active-response>
</ossec_config>
```

### **2.3. Filebeat Configuration**

**На siem-node1 - `/etc/filebeat/filebeat.yml`:**
```yaml
# Filebeat configuration
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/ossec/logs/alerts/alerts.json
  fields:
    log_type: wazuh_alerts
  json.keys_under_root: true
  json.overwrite_keys: true
  json.add_error_key: true

- type: log
  enabled: true
  paths:
    - /var/ossec/logs/archives/archives.json
  fields:
    log_type: wazuh_archives

# OpenSearch Output Configuration
output.opensearch:
  enabled: true
  hosts: ["https://192.168.10.10:9200", "https://192.168.10.11:9200"]
  protocol: "https"
  username: "admin"
  password: "admin"
  ssl:
    verification_mode: "none"
  indices:
    - index: "wazuh-alerts-%{+yyyy.MM.dd}"
      when.equals:
        fields.log_type: "wazuh_alerts"
    - index: "wazuh-archives-%{+yyyy.MM.dd}"
      when.equals:
        fields.log_type: "wazuh_archives"

# Setup
setup.template:
  name: "wazuh"
  pattern: "wazuh-*"
  overwrite: true
  enabled: true

setup.ilm:
  enabled: false

# Monitoring
monitoring:
  enabled: true
  period: 10s

# Logging
logging:
  level: info
  to_files: true
  files:
    path: /var/log/filebeat
    name: filebeat.log
    keepfiles: 7
    permissions: 0644
```

### **2.4. OpenSearch Dashboards Configuration**

**На siem-node3 - `/etc/opensearch-dashboards/opensearch_dashboards.yml`:**
```yaml
# OpenSearch Dashboards configuration
server.port: 5601
server.host: "192.168.10.12"
server.name: "siem-dashboard"
server.ssl.enabled: true
server.ssl.certificate: /etc/opensearch-dashboards/siem-node3.pem
server.ssl.key: /etc/opensearch-dashboards/siem-node3-key.pem

# OpenSearch connection
opensearch.hosts: ["https://192.168.10.10:9200", "https://192.168.10.11:9200"]
opensearch.ssl.verificationMode: none
opensearch.username: "admin"
opensearch.password: "admin"
opensearch.requestHeadersWhitelist: ["securitytenant", "Authorization", "osd-xsrf"]

# Security
opensearch_security.multitenancy.enabled: true
opensearch_security.multitenancy.tenants.preferred: ["Private", "Global"]
opensearch_security.readonly_mode.roles: ["kibana_read_only"]

# Wazuh plugin
opensearch_security.cookie.secure: true

# Internationalization
i18n.locale: "ru"

# Performance
opensearch.healthCheck.delay: 120000
opensearch.healthCheck.startupDelay: 120000

# Logging
logging:
  verbose: true
  dest: /var/log/opensearch-dashboards.log
  quiet: false
  timezone: UTC
```

---

## **ЧАСТЬ 3: Настройка агентов**

### **3.1. Конфигурация агента Linux**

**Файл: `/var/ossec/etc/ossec.conf` на агенте:**
```xml
<ossec_config>
  <client>
    <server>
      <address>192.168.10.10</address>
      <port>1514</port>
      <protocol>tcp</protocol>
      <queue_size>16384</queue_size>
    </server>
    <config_profile>linux, linux-server</config_profile>
    <notify_time>60</notify_time>
    <time-reconnect>300</time-reconnect>
    <auto_restart>yes</auto_restart>
  </client>

  <logging>
    <log_format>json</log_format>
  </logging>

  <!-- System Inventory -->
  <syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>
    <scan_on_start>yes</scan_on_start>
    
    <directories check_all="yes" realtime="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes" realtime="yes">/bin,/sbin</directories>
    
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/random.seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>
  </syscheck>

  <!-- Rootkit Detection -->
  <rootcheck>
    <disabled>no</disabled>
    <check_unixaudit>yes</check_unixaudit>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>
    <frequency>43200</frequency>
  </rootcheck>

  <!-- Log Monitoring -->
  <localfile>
    <location>/var/log/syslog</location>
    <log_format>syslog</log_format>
  </localfile>

  <localfile>
    <location>/var/log/auth.log</location>
    <log_format>syslog</log_format>
  </localfile>

  <!-- Active Response -->
  <active-response>
    <disabled>no</disabled>
    <ca_verification>yes</ca_verification>
  </active-response>
</ossec_config>
```

### **3.2. Регистрация агента**
```bash
# На агенте выполняем:
/var/ossec/bin/agent-auth -A "web-server-01" -m 192.168.10.10 -P "SecurePassword123!"

# Проверяем статус
systemctl status wazuh-agent
/var/ossec/bin/agent_control -l
```

---

## **ЧАСТЬ 4: Настройка системных служб и firewall**

### **4.1. Firewall Rules (UFW) на siem-node1**
```bash
# OpenSSH
sudo ufw allow from 192.168.0.0/16 to any port 22

# Wazuh Agents
sudo ufw allow from 192.168.0.0/16 to any port 1514
sudo ufw allow from 192.168.0.0/16 to any port 1515

# OpenSearch
sudo ufw allow from 192.168.10.0/24 to any port 9200
sudo ufw allow from 192.168.10.0/24 to any port 9300

# Filebeat
sudo ufw allow from 192.168.10.10 to any port 5044

# Enable firewall
sudo ufw enable
```

### **4.2. Systemd Services**

**OpenSearch Service - `/etc/systemd/system/opensearch.service`:**
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
LimitNOFILE=65536
LimitMEMLOCK=infinity

[Install]
WantedBy=multi-user.target
```

**Wazuh Manager Service - проверка конфигурации:**
```bash
systemctl cat wazuh-manager
# Output должен содержать:
# ExecStart=/var/ossec/bin/wazuh-manager
# Restart=on-failure
# RestartSec=10s
```

---

## **ЧАСТЬ 5: Проверка работоспособности**

### **5.1. Проверка соединений между компонентами**

```bash
# Проверка OpenSearch кластера
curl -XGET 'https://192.168.10.10:9200/_cluster/health?pretty' -u 'admin:admin' -k

# Проверка индексов
curl -XGET 'https://192.168.10.10:9200/_cat/indices?v' -u 'admin:admin' -k

# Проверка подключенных агентов
/var/ossec/bin/agent_control -l

# Проверка статуса Filebeat
systemctl status filebeat
journalctl -u filebeat -f

# Проверка логов Wazuh Manager
tail -f /var/ossec/logs/ossec.log
```

### **5.2. Мониторинг сети**
```bash
# Проверка открытых портов
netstat -tlnp | grep -E '(9200|9300|1514|1515|5601)'

# Проверка соединений
ss -tulpn | grep -E '(9200|9300|1514|1515|5601)'

# Мониторинг трафика
tcpdump -i ens192 port 1514 or port 9200 -n
```

### **5.3. Логи для диагностики**

**Wazuh Manager лог:**
```bash
tail -f /var/ossec/logs/ossec.log | grep -E "(ERROR|WARNING|connected)"
```

**OpenSearch лог:**
```bash
tail -f /var/log/opensearch/siem-cluster.log
```

**Filebeat лог:**
```bash
tail -f /var/log/filebeat/filebeat
```

---

## **ЧАСТЬ 6: Источники конфигураций и документация**

### **6.1. Официальная документация:**
- **Wazuh**: https://documentation.wazuh.com/current/
- **OpenSearch**: https://opensearch.org/docs/latest/
- **Filebeat**: https://www.elastic.co/guide/en/beats/filebeat/current/index.html

### **6.2. Критические файлы конфигурации:**
- `/var/ossec/etc/ossec.conf` - главный конфиг Wazuh
- `/etc/opensearch/opensearch.yml` - конфиг OpenSearch  
- `/etc/opensearch-dashboards/opensearch_dashboards.yml` - конфиг Dashboards
- `/etc/filebeat/filebeat.yml` - конфиг Filebeat

### **6.3. Полезные команды для диагностики:**
```bash
# Проверка всех служб
systemctl status opensearch wazuh-manager filebeat opensearch-dashboards

# Проверка логов в реальном времени
tail -f /var/ossec/logs/alerts/alerts.json | jq '.'

# Проверка нагрузки на OpenSearch
curl -XGET 'https://192.168.10.10:9200/_nodes/stats?pretty' -u 'admin:admin' -k

# Тестирование правил Wazuh
/var/ossec/bin/wazuh-logtest
```

Эта конфигурация обеспечит надежное взаимодействие всех компонентов SIEM системы с правильной маршрутизацией данных и отказоустойчивостью.

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

