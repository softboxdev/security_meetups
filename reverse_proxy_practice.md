# Подробная инструкция по установке и настройке обратного прокси сервера на Kali Linux (nginx)

## 1. Подготовка системы

### 1.1. Обновление системы
```bash
# Обновление списка пакетов
sudo apt update && sudo apt upgrade -y

# Установка необходимых зависимостей
sudo apt install -y curl wget vim gnupg lsb-release
```

### 1.2. Проверка сетевых настроек
```bash
# Проверка IP адреса
ip addr show

# Проверка доступности сети
ping -c 3 google.com

# Проверка открытых портов
ss -tlnp
```

## 2. Установка nginx

### 2.1. Установка из официальных репозиториев
```bash
# Установка nginx
sudo apt install -y nginx

# Проверка версии
nginx -v

# Проверка установки
systemctl status nginx
```

### 2.2. Настройка автозапуска
```bash
# Включение автозапуска
sudo systemctl enable nginx

# Запуск службы
sudo systemctl start nginx

# Проверка статуса
sudo systemctl status nginx
```

## 3. Базовая конфигурация nginx

### 3.1. Структура каталогов nginx
```bash
# Основные каталоги
/etc/nginx/           # Конфигурационные файлы
/var/www/html/        # Корневая директория веб-сервера
/var/log/nginx/       # Логи
/usr/share/nginx/     # Статические файлы
```

### 3.2. Тестирование базовой работы
```bash
# Проверка работы веб-сервера
curl -I http://localhost

# Проверка извне (если есть доступ)
curl -I http://your-server-ip
```

## 4. Настройка обратного прокси

### 4.1. Создание базовой конфигурации прокси

Создаем файл конфигурации для нашего прокси:
```bash
sudo nano /etc/nginx/sites-available/reverse-proxy
```

```nginx
# Основной конфигурационный файл обратного прокси
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # Логирование
    access_log /var/log/nginx/reverse-proxy.access.log;
    error_log /var/log/nginx/reverse-proxy.error.log;
    
    # Настройки безопасности
    server_tokens off;
    
    # Базовые настройки прокси
    location / {
        # Адрес целевого сервера
        proxy_pass http://localhost:8080;
        
        # Стандартные заголовки прокси
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Таймауты
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
        
        # Буферизация
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        
        # Отключение проверки сертификатов для внутренних сервисов
        proxy_ssl_verify off;
    }
    
    # Запрет доступа к скрытым файлам
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

### 4.2. Активация конфигурации

```bash
# Создание симлинка
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/

# Удаление дефолтного сайта (опционально)
sudo rm /etc/nginx/sites-enabled/default

# Проверка синтаксиса конфигурации
sudo nginx -t

# Перезагрузка nginx
sudo systemctl reload nginx
```

## 5. Расширенные конфигурации прокси

### 5.1. Прокси для нескольких бэкенд-серверов

```bash
sudo nano /etc/nginx/sites-available/multi-backend-proxy
```

```nginx
# Конфигурация для нескольких бэкендов
upstream backend_servers {
    # Балансировка нагрузки
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080 weight=1;
    
    # Настройки балансировки
    least_conn;
    keepalive 32;
}

server {
    listen 80;
    server_name api.your-domain.com;
    
    location / {
        proxy_pass http://backend_servers;
        
        # Заголовки для бэкенда
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;
        
        # Health check
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    }
}
```

### 5.2. Прокси с SSL терминацией

```bash
sudo nano /etc/nginx/sites-available/ssl-proxy
```

```nginx
# HTTPS прокси с SSL терминацией
server {
    listen 443 ssl http2;
    server_name secure.your-domain.com;
    
    # Пути к SSL сертификатам
    ssl_certificate /etc/ssl/certs/your-domain.crt;
    ssl_certificate_key /etc/ssl/private/your-domain.key;
    
    # Настройки SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    
    location / {
        proxy_pass http://backend_servers;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name secure.your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

### 5.3. Прокси с аутентификацией

```bash
sudo nano /etc/nginx/sites-available/auth-proxy
```

```nginx
# Прокси с базовой аутентификацией
server {
    listen 80;
    server_name admin.your-domain.com;
    
    # Базовая аутентификация
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    location / {
        proxy_pass http://backend_admin_panel;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Передача заголовка аутентификации
        proxy_set_header Authorization $http_authorization;
        proxy_pass_header Authorization;
    }
}
```

Создание файла с пользователями:
```bash
# Установка утилиты для создания паролей
sudo apt install -y apache2-utils

# Создание файла с пользователями
sudo htpasswd -c /etc/nginx/.htpasswd username1
sudo htpasswd /etc/nginx/.htpasswd username2
```

## 6. Оптимизация производительности

### 6.1. Настройка основного конфига nginx

```bash
sudo nano /etc/nginx/nginx.conf
```

```nginx
user www-data;
worker_processes auto;
worker_rlimit_nofile 100000;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

http {
    # Базовые настройки
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Кэширование файловых дескрипторов
    open_file_cache max=200000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
    
    # Буферизация
    client_body_buffer_size 128k;
    client_max_body_size 100m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    output_buffers 1 32k;
    postpone_output 1460;
    
    # Таймауты
    client_header_timeout 30s;
    client_body_timeout 30s;
    send_timeout 30s;
    
    # Gzip сжатие
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    # Настройки прокси по умолчанию
    proxy_connect_timeout 30s;
    proxy_send_timeout 30s;
    proxy_read_timeout 30s;
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
    
    # Включение конфигураций сайтов
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## 7. Мониторинг и логирование

### 7.1. Настройка расширенного логирования

```bash
sudo nano /etc/nginx/conf.d/logging.conf
```

```nginx
# Формат лога с дополнительной информацией о прокси
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for" '
                'proxy: $upstream_addr $upstream_status $upstream_response_time';
                
log_format proxy_log '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" '
                     'proxy: $proxy_host $upstream_addr '
                     'rt=$request_time uct="$upstream_connect_time" '
                     'uht="$upstream_header_time" urt="$upstream_response_time"';

access_log /var/log/nginx/access.log main;
```

### 7.2. Статус страница nginx

```bash
sudo nano /etc/nginx/conf.d/status.conf
```

```nginx
# Статус страница для мониторинга
server {
    listen 127.0.0.1:8080;
    server_name localhost;
    
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
    
    location /server_status {
        # Расширенная статистика
        return 200 "Active connections: $connections_active\n"
                  "Reading: $connections_reading\n"
                  "Writing: $connections_writing\n"
                  "Waiting: $connections_waiting\n";
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

## 8. Безопасность

### 8.1. Базовая защита

```bash
sudo nano /etc/nginx/conf.d/security.conf
```

```nginx
# Базовые настройки безопасности
# Защита от распространенных атак
server {
    # Запрет нежелательных User-Agent
    if ($http_user_agent ~* (wget|curl|nikto|sqlmap|nmap) ) {
        return 403;
    }
    
    # Ограничение методов HTTP
    if ($request_method !~ ^(GET|HEAD|POST)$ ) {
        return 405;
    }
    
    # Защита от ботов и сканеров
    location ~* (\.htaccess|\.htpasswd|\.git|\.env) {
        deny all;
        access_log off;
        log_not_found off;
    }
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=general:10m rate=1r/s;

# Заголовки безопасности
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### 8.2. Настройка firewall

```bash
# Настройка UFW
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status verbose
```

## 9. Тестирование и отладка

### 9.1. Команды для тестирования

```bash
# Проверка синтаксиса конфигурации
sudo nginx -t

# Проверка загруженной конфигурации
sudo nginx -T

# Перезагрузка конфигурации
sudo systemctl reload nginx

# Полная перезагрузка службы
sudo systemctl restart nginx

# Просмотр логов в реальном времени
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Проверка открытых портов
sudo netstat -tlnp | grep nginx
ss -tlnp | grep nginx

# Тестирование прокси
curl -I http://your-domain.com
curl -X GET http://your-domain.com/api/test
```

### 9.2. Создание тестового бэкенд-сервера

```bash
# Установка Python для тестового сервера
sudo apt install -y python3

# Создание простого тестового сервера
cat > /tmp/test_backend.py << 'EOF'
from http.server import HTTPServer, BaseHTTPRequestHandler

class SimpleHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-Type', 'text/plain')
        self.end_headers()
        response = f"Hello from backend! Path: {self.path}\nHeaders: {self.headers}"
        self.wfile.write(response.encode())
    
    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        self.send_response(200)
        self.send_header('Content-Type', 'text/plain')
        self.end_headers()
        response = f"Received POST data: {post_data.decode()}"
        self.wfile.write(response.encode())

if __name__ == '__main__':
    server = HTTPServer(('localhost', 8080), SimpleHandler)
    print("Starting test backend on port 8080...")
    server.serve_forever()
EOF

# Запуск тестового сервера в фоне
python3 /tmp/test_backend.py &
```

## 10. Полезные скрипты для управления

### 10.1. Скрипт для быстрого управления

```bash
#!/bin/bash
# /usr/local/bin/nginx-manager.sh

case "$1" in
    start)
        sudo systemctl start nginx
        echo "Nginx started"
        ;;
    stop)
        sudo systemctl stop nginx
        echo "Nginx stopped"
        ;;
    restart)
        sudo systemctl restart nginx
        echo "Nginx restarted"
        ;;
    reload)
        sudo systemctl reload nginx
        echo "Nginx configuration reloaded"
        ;;
    status)
        sudo systemctl status nginx
        ;;
    test)
        sudo nginx -t
        ;;
    logs)
        sudo tail -f /var/log/nginx/error.log
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|reload|status|test|logs}"
        exit 1
        ;;
esac
```

Сделать скрипт исполняемым:
```bash
sudo chmod +x /usr/local/bin/nginx-manager.sh
```

## 11. Финальная проверка

### 11.1. Полная проверка системы

```bash
# Проверка всех конфигураций
echo "=== Testing nginx configuration ==="
sudo nginx -t

echo "=== Checking nginx status ==="
sudo systemctl status nginx

echo "=== Checking open ports ==="
sudo ss -tlnp | grep nginx

echo "=== Checking logs for errors ==="
sudo tail -20 /var/log/nginx/error.log

echo "=== Testing proxy functionality ==="
curl -I http://localhost
```

### 11.2. Рекомендации по эксплуатации

1. **Регулярное обновление**: `sudo apt update && sudo apt upgrade`
2. **Мониторинг логов**: Настройка logrotate и мониторинг ошибок
3. **Резервное копирование**: Конфигурационных файлов
4. **SSL сертификаты**: Использование Let's Encrypt для автоматического обновления
5. **Безопасность**: Регулярный аудит конфигураций и обновление правил безопасности

Поздравляю у вас настроен полнофункциональный обратный прокси-сервер на Kali Linux с nginx, готовый к использованию в production-среде!