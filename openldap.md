# OpenLDAP: подробное руководство с диаграммами последовательностей

## 1. Что такое OpenLDAP?

### **Основные понятия**

**OpenLDAP** — это открытая реализация протокола LDAP (Lightweight Directory Access Protocol), представляющая собой иерархическую базу данных для хранения и управления информацией о пользователях, группах, устройствах и других объектах в сети.

### **Архитектура OpenLDAP**

```mermaid
graph TB
    A[Клиентские приложения] --> B[OpenLDAP Сервер]
    B --> C[Backend База данных]
    B --> D[Служба репликации]
    B --> E[Модули расширений]
    
    F[Административные инструменты] --> B
    G[Внешние системы аутентификации] --> B
    
    subgraph "Компоненты OpenLDAP"
        C
        H[Slapd - демон LDAP]
        I[Slapd.conf - конфигурация]
        J[Схемы данных]
    end
    
    B --> H
    H --> I
    H --> J
```

### **Ключевые компоненты**

#### **Slapd (Stand-alone LDAP Daemon)**
- Основной демон, обрабатывающий LDAP-запросы
- Управляет подключениями, аутентификацией и операциями с данными

#### **Backend базы данных**
- **BDB/HDB:** Berkeley DB для хранения данных
- **MDB:** Memory-Mapped DB (современная, более производительная)
- **LDAP:** Прокси к другим LDAP-серверам

#### **Схемы данных (Schemas)**
- Определяют структуру и типы данных
- Стандартные схемы: core, cosine, inetorgperson, nis

### **DIT (Directory Information Tree)**

```mermaid
graph TD
    A[dc=company,dc=com] --> B[ou=People]
    A --> C[ou=Groups]
    A --> D[ou=Computers]
    
    B --> E[uid=john,ou=People]
    B --> F[uid=jane,ou=People]
    
    C --> G[cn=Admins,ou=Groups]
    C --> H[cn=Users,ou=Groups]
    
    E --> I[Атрибуты:<br/>cn: John Doe<br/>mail: john@company.com]
    F --> J[Атрибуты:<br/>cn: Jane Smith<br/>mail: jane@company.com]
    
    G --> K[Члены:<br/>uid=john]
```

## 2. LDAP запросы

### **Структура LDAP запроса**

```mermaid
sequenceDiagram
    participant C as LDAP Клиент
    participant S as OpenLDAP Сервер
    
    Note over C,S: Установление соединения
    C->>S: TCP подключение порт 389/636
    
    Note over C,S: Привязка (Bind) к серверу
    C->>S: bind(dn, password)
    S->>C: bind_success
    
    Note over C,S: Выполнение операций
    C->>S: search(base_dn, filter, attributes)
    S->>S: Поиск в DIT
    S->>C: search_result
    
    C->>S: unbind()
    S->>C: Соединение закрыто
```

### **Типы LDAP операций**

#### **Операции поиска (Search)**
```ldif
# Базовый поиск
ldapsearch -x -b "dc=company,dc=com" "(objectClass=*)"

# Поиск с фильтром
ldapsearch -x -b "ou=People,dc=company,dc=com" "(uid=john*)"

# Поиск определенных атрибутов
ldapsearch -x -b "dc=company,dc=com" "(cn=John*)" cn mail telephoneNumber
```

#### **Операции модификации**
```ldif
# Добавление записи
dn: uid=newuser,ou=People,dc=company,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: top
cn: New User
sn: User
uid: newuser
userPassword: {SSHA}hashedpassword

# Изменение записи
dn: uid=john,ou=People,dc=company,dc=com
changetype: modify
replace: mail
mail: john.new@company.com

# Удаление записи
dn: uid=olduser,ou=People,dc=company,dc=com
changetype: delete
```

### **LDAP фильтры**

#### **Базовые операторы фильтров**
```ldif
# Равенство
(cn=John Doe)
(uid=john*)

# Логические операторы
(&(objectClass=person)(cn=John*))
(|(department=IT)(department=Sales))
(!(status=disabled))

# Присутствие атрибута
(telephoneNumber=*)
(mail=*@company.com)

# Диапазоны
(uidNumber>=1000)
(createTimestamp<=20240101000000Z)
```

#### **Сложные фильтры**
```ldif
# Поиск пользователей в определенном отделе с email
(&(objectClass=inetOrgPerson)
  (department=IT)
  (mail=*@company.com)
  (!(employeeType=contractor)))

# Поиск групп, содержащих пользователя
(&(objectClass=groupOfNames)
  (member=uid=john,ou=People,dc=company,dc=com))
```

### **Процесс выполнения LDAP запроса**

```mermaid
sequenceDiagram
    participant C as Клиент
    participant F as Frontend
    participant B as Backend DB
    participant I as Index
    
    C->>F: LDAP Search Request
    F->>F: Проверка ACL
    F->>F: Парсинг фильтра
    
    alt Индекс существует
        F->>I: Поиск по индексу
        I->>F: Список DN
    else Индекс отсутствует
        F->>B: Полный перебор
        B->>F: Все записи
    end
    
    F->>F: Применение фильтра
    F->>F: Ограничение атрибутов
    F->>C: Search Result Entry
    F->>C: Search Result Done
```

## 3. Аутентификация OpenLDAP

### **Методы аутентификации**

```mermaid
graph LR
    A[Методы аутентификации] --> B[Простая bind]
    A --> C[SASL механизмы]
    A --> D[Client Certificate]
    
    B --> B1[DN + пароль]
    B --> B2[Анонимный доступ]
    
    C --> C1[DIGEST-MD5]
    C --> C2[GSSAPI Kerberos]
    C --> C3[EXTERNAL]
    
    D --> D1[TLS сертификаты]
```

### **Процесс простой аутентификации (Simple Bind)**

```mermaid
sequenceDiagram
    participant U as Пользователь
    participant A as Приложение
    participant L as OpenLDAP
    participant D as База данных
    
    U->>A: Ввод логина/пароля
    A->>L: bind(dn=user_dn, password)
    
    L->>D: Поиск записи пользователя
    D->>L: user_entry
    
    L->>L: Проверка userPassword
    alt Пароль верный
        L->>A: bind_success
        A->>U: Успешный вход
        A->>L: search(base=user_dn, scope=base)
        L->>A: user_attributes
    else Пароль неверный
        L->>A: invalidCredentials
        A->>U: Ошибка аутентификации
    end
```

### **Поиск DN пользователя для аутентификации**

```mermaid
sequenceDiagram
    participant U as Пользователь
    participant A as Приложение
    participant L as OpenLDAP
    
    U->>A: Ввод username/password
    A->>L: Поиск DN пользователя
    
    Note over A,L: Анонимный поиск для определения DN
    A->>L: bind(anonymous)
    L->>A: bind_success
    
    A->>L: search(base="ou=People,dc=company,dc=com",<br/>filter="(uid=username)")
    L->>A: user_dn
    
    A->>L: bind(dn=user_dn, password=user_password)
    alt Успех
        L->>A: bind_success
        A->>U: Аутентификация успешна
    else Неудача
        L->>A: invalidCredentials
        A->>U: Ошибка аутентификации
    end
```

### **SASL аутентификация**

```mermaid
sequenceDiagram
    participant U as Пользователь
    participant A as Приложение
    participant L as OpenLDAP
    participant K as KDC Kerberos
    
    U->>A: Запрос аутентификации
    A->>K: Запрос билета Kerberos
    K->>A: TGT билет
    
    A->>L: SASL Bind с GSSAPI
    L->>A: SASL challenge
    A->>K: Проверка билета
    K->>A: Подтверждение
    
    A->>L: SASL response
    L->>L: Проверка подлинности
    L->>A: bind_success
    A->>U: Успешный вход
```

### **Настройка аутентификации в конфигурации**

#### **slapd.conf основные параметры**
```
# Базовая конфигурация
database mdb
suffix "dc=company,dc=com"
rootdn "cn=admin,dc=company,dc=com"
rootpw {SSHA}hashedpassword

# Политики паролей
password-hash {SSHA}
password-min-length 8
password-check-quality on

# ACL для аутентификации
access to attrs=userPassword
    by self write
    by anonymous auth
    by * none

access to *
    by self write
    by users read
    by anonymous auth
```

### **Поиск пользователя и групп**

```mermaid
sequenceDiagram
    participant A as Приложение
    participant L as OpenLDAP
    
    Note over A,L: Фаза 1: Поиск пользователя
    A->>L: search(base="ou=People,dc=com",<br/>filter="(uid=john)")
    L->>A: user_dn: uid=john,ou=People,dc=com
    
    Note over A,L: Фаза 2: Аутентификация
    A->>L: bind(dn=user_dn, password)
    L->>A: bind_success
    
    Note over A,L: Фаза 3: Поиск групп пользователя
    A->>L: search(base="ou=Groups,dc=com",<br/>filter="(member=user_dn)")
    L->>A: group_cn: cn=admins,ou=Groups,dc=com
    
    Note over A,L: Фаза 4: Проверка членства
    A->>L: compare(dn=group_dn,<br/>attr: member, value: user_dn)
    L->>A: compare_true
```

### **Обработка ошибок аутентификации**

```mermaid
sequenceDiagram
    participant U as Пользователь
    participant A as Приложение
    participant L как OpenLDAP
    participant M как Monitoring
    
    U->>A: Попытка входа
    A->>L: bind(dn, wrong_password)
    L->>A: invalidCredentials (49)
    
    A->>A: Увеличение счетчика ошибок
    alt Превышено максимальное количество попыток
        A->>L: modify(dn, replace: pwdAccountLockedTime)
        L->>A: modify_success
        A->>M: Логирование блокировки
        A->>U: Учетная запись заблокирована
    else В пределах лимита
        A->>U: Неверный пароль
    end
```

## 4. Практические примеры использования

### **Интеграция с PAM**
```
# /etc/pam.d/system-auth
auth sufficient pam_ldap.so
account sufficient pam_ldap.so
password sufficient pam_ldap.so
session optional pam_ldap.so
```

### **Интеграция с SSH**
```
# /etc/ssh/sshd_config
PubkeyAuthentication yes
PasswordAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
LDAPURI ldap://ldap.company.com
LDAPBaseDC ou=People,dc=company,dc=com
```

### **Web-приложения**
```
# PHP LDAP аутентификация
$ldapconn = ldap_connect("ldap.company.com");
ldap_set_option($ldapconn, LDAP_OPT_PROTOCOL_VERSION, 3);

$user_dn = "uid=" . $username . ",ou=People,dc=company,dc=com";
$auth = ldap_bind($ldapconn, $user_dn, $password);

if ($auth) {
    // Пользователь аутентифицирован
    $search = ldap_search($ldapconn, $user_dn, "(objectClass=*)");
    $user_info = ldap_get_entries($ldapconn, $search);
}
```

