
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