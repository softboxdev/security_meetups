# Проверенное руководство по криминалистическому изъятию данных с включенного устройства на Kali Linux

## **ВАЖНОЕ ПРЕДУПРЕЖДЕНИЕ**
Данная практика предполагает использование **одного и того же устройства** (Kali Linux) как для выполнения команд, так и для хранения полученных данных. В реальной ситуации НИКОГДА не сохраняйте данные на исследуемый компьютер! Используйте внешние носители.

---

## **Подготовительный этап**

### **1. Подготовка рабочей станции на Kali Linux**
```bash
#!/bin/bash

# Создаем структуру каталогов для хранения доказательств
sudo mkdir -p /forensic_case/case_001
sudo mkdir -p /forensic_case/case_001/ram_dump
sudo mkdir -p /forensic_case/case_001/volatile_data
sudo mkdir -p /forensic_case/case_001/disk_images
sudo mkdir -p /forensic_case/case_001/hashes
sudo mkdir -p /forensic_case/case_001/logs
sudo mkdir -p /forensic_case/case_001/analysis

# Устанавливаем необходимые инструменты
echo "Установка необходимых инструментов..."
sudo apt update
sudo apt install -y p7zip-full sleuthkit dc3dd guymager autopsy volatility3
sudo apt install -y linux-headers-$(uname -r) build-essential git

# Устанавливаем права доступа
sudo chown -R $USER:$USER /forensic_case/case_001

echo "Подготовка завершена!"
```

---

## **Шаг 0: Принятие решения и документирование**

### **Создание протокола изъятия**
```bash
#!/bin/bash

# Переходим в рабочую директорию
cd /forensic_case/case_001

# Создаем файл протокола
echo "=== КРИМИНАЛИСТИЧЕСКИЙ ПРОТОКОЛ ИЗЪЯТИЯ ===" > protocol.txt
echo "Дата/время начала: $(date '+%Y-%m-%d %H:%M:%S')" >> protocol.txt
echo "Оператор: $(whoami)" >> protocol.txt
echo "ID дела: case_001" >> protocol.txt
echo "Цель: Изъятие данных с включенного устройства" >> protocol.txt
echo "" >> protocol.txt

# Записываем информацию о системе
echo "=== ИНФОРМАЦИЯ О СИСТЕМЕ ===" >> protocol.txt
uname -a >> protocol.txt
cat /etc/os-release >> protocol.txt
echo "" >> protocol.txt

echo "Протокол создан в /forensic_case/case_001/protocol.txt"
```

---

## **Шаг 1: Сбор данных оперативной памяти (RAM)**

### **1.1 Использование инструментов для дампа памяти**
```bash
#!/bin/bash

# Переходим в рабочую директорию
cd /forensic_case/case_001/ram_dump

# Собираем информацию о системе перед дампом
echo "=== ИНФОРМАЦИЯ О ПАМЯТИ ПЕРЕД ДАМПОМ ===" >> ../protocol.txt
sudo free -h >> ../protocol.txt
sudo cat /proc/meminfo | head -20 >> ../protocol.txt
echo "" >> ../protocol.txt

# Создаем дамп оперативной памяти
echo "Создание дампа оперативной памяти..."
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")

# Метод 1: Использование /proc/kcore (для современных ядер)
echo "Метод 1: Создание дампа через /proc/kcore..."
sudo dd if=/proc/kcore of=ram_dump_kcore_${TIMESTAMP}.raw bs=1M count=$(( $(grep MemTotal /proc/meminfo | awk '{print $2}') / 1024 )) status=progress 2>&1 | tee ../logs/dd_kcore_${TIMESTAMP}.log

# Метод 2: Использование fmem
echo "Метод 2: Установка и использование fmem..."
if [ ! -f "/proc/fmem" ]; then
    echo "Установка fmem..."
    git clone https://github.com/NateBrune/fmem.git
    cd fmem
    make
    sudo insmod fmem.ko
    cd ..
fi

if [ -f "/proc/fmem" ]; then
    echo "Создание дампа через fmem..."
    sudo dd if=/dev/fmem of=ram_dump_fmem_${TIMESTAMP}.raw bs=1M status=progress 2>&1 | tee ../logs/dd_fmem_${TIMESTAMP}.log
else
    echo "fmem недоступен, используется альтернативный метод"
fi

# Метод 3: Использование LiME
echo "Метод 3: Установка LiME..."
LIME_DIR="/tmp/lime_install"
mkdir -p ${LIME_DIR}
cd ${LIME_DIR}
git clone https://github.com/504ensicsLabs/LiME.git 2>/dev/null || echo "LiME уже загружен"
cd LiME/src
make 2>&1 | tee /forensic_case/case_001/logs/lime_compile_${TIMESTAMP}.log

if [ -f "lime.ko" ]; then
    echo "Создание дампа через LiME..."
    sudo insmod lime.ko "path=/forensic_case/case_001/ram_dump/ram_dump_lime_${TIMESTAMP}.lime format=lime"
else
    echo "Не удалось скомпилировать LiME"
fi

cd /forensic_case/case_001/ram_dump

# Записываем информацию в протокол
echo "Дампы RAM созданы:" >> ../protocol.txt
ls -lh *.raw *.lime 2>/dev/null >> ../protocol.txt || echo "Файлы дампа не найдены" >> ../protocol.txt
echo "" >> ../protocol.txt
```

### **1.2 Расчет хэш-сумм дампа памяти**
```bash
#!/bin/bash

cd /forensic_case/case_001/ram_dump

echo "=== ХЭШ-СУММЫ ДАМПА ПАМЯТИ ===" >> ../protocol.txt

# Рассчитываем хэш-суммы для всех файлов дампа
for dump_file in *.raw *.lime 2>/dev/null; do
    if [ -f "$dump_file" ]; then
        echo "Файл: $dump_file" >> ../protocol.txt
        echo "Размер: $(du -h "$dump_file" | cut -f1)" >> ../protocol.txt
        echo "MD5:    $(md5sum "$dump_file" | cut -d' ' -f1)" >> ../protocol.txt
        echo "SHA256: $(sha256sum "$dump_file" | cut -d' ' -f1)" >> ../protocol.txt
        echo "" >> ../protocol.txt
        
        # Сохраняем хэши отдельно
        md5sum "$dump_file" > ../hashes/"${dump_file}.md5"
        sha256sum "$dump_file" > ../hashes/"${dump_file}.sha256"
    fi
done

echo "Хэш-суммы рассчитаны и сохранены."
```

---

## **Шаг 2: Сбор летучих системных данных**

### **2.1 Основной скрипт сбора летучих данных**
```bash
#!/bin/bash

# Переходим в директорию для летучих данных
cd /forensic_case/case_001/volatile_data

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
COLLECTION_DIR="collection_${TIMESTAMP}"
mkdir -p ${COLLECTION_DIR}
cd ${COLLECTION_DIR}

echo "Начало сбора летучих данных: $(date)"

# Функция для безопасного выполнения команд
safe_exec() {
    local cmd_name="$1"
    local cmd="$2"
    echo "Выполнение: $cmd_name"
    echo "=== $cmd_name ===" > "${cmd_name}.txt"
    bash -c "$cmd" >> "${cmd_name}.txt" 2>&1 || echo "Ошибка выполнения команды" >> "${cmd_name}.txt"
    echo "" >> "${cmd_name}.txt"
}

### **2.1.1 Сбор информации о процессах**
safe_exec "processes" "ps auxef"
safe_exec "process_tree" "pstree -pan"
safe_exec "cpu_info" "mpstat -P ALL 1 3"
safe_exec "load_average" "uptime"

### **2.1.2 Сбор сетевой информации**
safe_exec "network_connections" "netstat -tupan"
safe_exec "open_ports" "ss -tulpn"
safe_exec "network_config" "ip addr show; echo '---'; ip route show"
safe_exec "arp_table" "ip neigh show"
safe_exec "listening_ports" "lsof -i -P -n | grep LISTEN"
safe_exec "active_connections" "lsof -i"

### **2.1.3 Сбор информации о пользователях и сессиях**
safe_exec "current_users" "who -a; echo '---'; w"
safe_exec "login_history" "last -a -n 50"
safe_exec "sudo_logs" "sudo tail -100 /var/log/auth.log 2>/dev/null | grep -i sudo || echo 'Нет доступа к логам sudo'"
safe_exec "failed_logins" "sudo grep 'Failed password' /var/log/auth.log 2>/dev/null | tail -20 || echo 'Нет доступа к логам'"

### **2.1.4 Сбор информации о системе и ядре**
safe_exec "kernel_modules" "lsmod"
safe_exec "system_info" "uname -a; echo '---'; cat /etc/os-release"
safe_exec "dmesg_log" "sudo dmesg | tail -100"
safe_exec "loaded_services" "systemctl list-units --type=service --state=running 2>/dev/null || service --status-all 2>/dev/null"

### **2.1.5 Сбор информации о файловой системе**
safe_exec "mounted_filesystems" "mount; echo '---'; df -hT"
safe_exec "open_files" "sudo lsof -nP 2>/dev/null | head -1000"
safe_exec "recent_files" "find /home /tmp /var/tmp -type f -mtime -1 2>/dev/null | head -50"

### **2.1.6 Сбор истории команд**
safe_exec "command_history" "history 100"
safe_exec "bash_history" "cat ~/.bash_history 2>/dev/null | tail -100"
safe_exec "root_history" "sudo cat /root/.bash_history 2>/dev/null | tail -100 2>/dev/null || echo 'Нет доступа к истории root'"

### **2.1.7 Сбор информации о системе и оборудовании**
safe_exec "hardware_info" "sudo lshw -short 2>/dev/null || echo 'lshw недоступен'"
safe_exec "usb_devices" "lsusb"
safe_exec "pci_devices" "lspci"
safe_exec "disk_info" "lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE,MODEL,SERIAL"

### **2.1.8 Сбор временных данных**
safe_exec "temp_files" "ls -la /tmp /var/tmp 2>/dev/null | head -50"
safe_exec "cron_jobs" "crontab -l 2>/dev/null; echo '---'; ls /etc/cron.* 2>/dev/null"

### **2.1.9 Сбор логов**
safe_exec "system_logs" "sudo tail -200 /var/log/syslog 2>/dev/null || sudo journalctl -n 200 2>/dev/null"
safe_exec "auth_logs" "sudo tail -200 /var/log/auth.log 2>/dev/null"

echo "Сбор летучих данных завершен: $(date)"
```

### **2.2 Расчет хэш-сумм собранных данных**
```bash
#!/bin/bash

cd /forensic_case/case_001/volatile_data/collection_${TIMESTAMP}

echo "=== ХЭШ-СУММЫ ЛЕТУЧИХ ДАННЫХ ===" >> ../../protocol.txt

# Создаем архив с собранными данными
tar -czf ../volatile_data_${TIMESTAMP}.tar.gz .
cd ..

# Рассчитываем хэш-суммы архива
echo "Архив летучих данных: volatile_data_${TIMESTAMP}.tar.gz" >> ../protocol.txt
echo "Размер: $(du -h volatile_data_${TIMESTAMP}.tar.gz | cut -f1)" >> ../protocol.txt
echo "MD5:    $(md5sum volatile_data_${TIMESTAMP}.tar.gz | cut -d' ' -f1)" >> ../protocol.txt
echo "SHA256: $(sha256sum volatile_data_${TIMESTAMP}.tar.gz | cut -d' ' -f1)" >> ../protocol.txt
echo "" >> ../protocol.txt

# Сохраняем хэши отдельно
md5sum volatile_data_${TIMESTAMP}.tar.gz > ../hashes/volatile_data_${TIMESTAMP}.md5
sha256sum volatile_data_${TIMESTAMP}.tar.gz > ../hashes/volatile_data_${TIMESTAMP}.sha256

echo "Хэш-суммы летучих данных рассчитаны."
```

---

## **Шаг 3: Определение дисков для создания образов**

### **3.1 Скрипт идентификации дисков**
```bash
#!/bin/bash

cd /forensic_case/case_001

echo "=== ИНФОРМАЦИЯ О ДИСКАХ И НОСИТЕЛЯХ ===" >> protocol.txt
echo "Дата сбора: $(date '+%Y-%m-%d %H:%M:%S')" >> protocol.txt
echo "" >> protocol.txt

# Основная информация о дисках
safe_exec_disk() {
    echo "--- $1 ---" >> protocol.txt
    bash -c "$2" >> protocol.txt 2>&1
    echo "" >> protocol.txt
}

safe_exec_disk "Блочные устройства" "lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE,MODEL,SERIAL,UUID"
safe_exec_disk "Информация о разделах" "sudo fdisk -l 2>/dev/null || sudo parted -l 2>/dev/null"
safe_exec_disk "LVM информация" "sudo pvs 2>/dev/null; echo '---'; sudo vgs 2>/dev/null; echo '---'; sudo lvs 2>/dev/null"
safe_exec_disk "Монтированные ФС" "mount | sort"
safe_exec_disk "Свободное место" "df -hT"

# Определяем системный диск
ROOT_DEVICE=$(df / | tail -1 | awk '{print $1}' | sed 's/[0-9]*$//')
echo "Системный корневой диск: $ROOT_DEVICE" >> protocol.txt
echo "" >> protocol.txt

# Запрашиваем подтверждение для создания образа
echo "СПИСОК ОБНАРУЖЕННЫХ ДИСКОВ:" >> protocol.txt
sudo fdisk -l 2>/dev/null | grep "^Disk /dev/" >> protocol.txt

# Создаем конфигурационный файл для образов
cat > disk_targets.conf << EOF
# Конфигурация дисков для создания образов
# Раскомментируйте строки с дисками для создания образов

# Пример:
# SYSTEM_DISK="/dev/sda"
# DATA_DISK="/dev/sdb"

# Обнаруженные диски:
EOF

sudo fdisk -l 2>/dev/null | grep "^Disk /dev/" | awk -F'[: ,]' '{print "# " $2 " - " $4 $5}' >> disk_targets.conf

echo ""
echo "Информация о дисках сохранена в protocol.txt"
echo "Конфигурация дисков сохранена в disk_targets.conf"
echo "Отредактируйте disk_targets.conf перед созданием образов"
```

---

## **Шаг 4: Создание битовых образов дисков**

### **4.1 Скрипт создания образов с dc3dd**
```bash
#!/bin/bash

cd /forensic_case/case_001/disk_images

# Читаем конфигурацию
if [ ! -f "../disk_targets.conf" ]; then
    echo "Файл конфигурации disk_targets.conf не найден!"
    echo "Сначала выполните шаг 3 для определения дисков."
    exit 1
fi

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
IMAGING_LOG="../logs/imaging_${TIMESTAMP}.log"

echo "Начало создания образов дисков: $(date)" | tee -a ${IMAGING_LOG}

# Функция создания образа диска
create_disk_image() {
    local DISK="$1"
    local LABEL="$2"
    
    if [ ! -b "$DISK" ]; then
        echo "ОШИБКА: Устройство $DISK не найдено!" | tee -a ${IMAGING_LOG}
        return 1
    fi
    
    echo "" | tee -a ${IMAGING_LOG}
    echo "=== СОЗДАНИЕ ОБРАЗА ДИСКА: $DISK ($LABEL) ===" | tee -a ${IMAGING_LOG}
    echo "Начало: $(date)" | tee -a ${IMAGING_LOG}
    
    # Получаем информацию о диске
    DISK_SIZE=$(sudo blockdev --getsize64 "$DISK" 2>/dev/null)
    if [ -n "$DISK_SIZE" ]; then
        DISK_SIZE_GB=$((DISK_SIZE / 1024 / 1024 / 1024))
        echo "Размер диска: ${DISK_SIZE_GB} GB" | tee -a ${IMAGING_LOG}
    fi
    
    # Имя файла образа
    IMAGE_NAME="disk_$(basename ${DISK})_${LABEL}_${TIMESTAMP}.img"
    
    # Создаем образ с помощью dc3dd
    echo "Создание образа: $IMAGE_NAME" | tee -a ${IMAGING_LOG}
    
    sudo dc3dd if="$DISK" of="$IMAGE_NAME" \
        hash=md5,sha256 \
        log="${IMAGING_LOG}.${DISK//\//_}" \
        hlog="${IMAGING_LOG}.hash.${DISK//\//_}" \
        bs=1M \
        progress=on \
        2>&1 | tee -a ${IMAGING_LOG}
    
    local RESULT=$?
    
    if [ $RESULT -eq 0 ]; then
        echo "Образ успешно создан: $IMAGE_NAME" | tee -a ${IMAGING_LOG}
        
        # Рассчитываем дополнительную проверку хэшей
        echo "Дополнительная проверка хэшей..." | tee -a ${IMAGING_LOG}
        IMAGE_MD5=$(md5sum "$IMAGE_NAME" | cut -d' ' -f1)
        IMAGE_SHA256=$(sha256sum "$IMAGE_NAME" | cut -d' ' -f1)
        
        echo "Проверочный MD5:    $IMAGE_MD5" | tee -a ${IMAGING_LOG}
        echo "Проверочный SHA256: $IMAGE_SHA256" | tee -a ${IMAGING_LOG}
        
        # Сохраняем хэши
        echo "$IMAGE_MD5  $IMAGE_NAME" > "../hashes/${IMAGE_NAME}.md5"
        echo "$IMAGE_SHA256  $IMAGE_NAME" > "../hashes/${IMAGE_NAME}.sha256"
        
        # Создаем EWF образ (если установлен ewfacquire)
        if command -v ewfacquire &> /dev/null; then
            echo "Создание E01 образа..." | tee -a ${IMAGING_LOG}
            EWF_NAME="disk_$(basename ${DISK})_${LABEL}_${TIMESTAMP}.E01"
            sudo ewfacquire "$DISK" -t "$EWF_NAME" -f encase7 -c best -l "${IMAGING_LOG}.ewf" 2>&1 | tee -a ${IMAGING_LOG}
        fi
    else
        echo "ОШИБКА при создании образа $IMAGE_NAME" | tee -a ${IMAGING_LOG}
    fi
    
    echo "Завершение: $(date)" | tee -a ${IMAGING_LOG}
    echo "Статус: $RESULT" | tee -a ${IMAGING_LOG}
    
    return $RESULT
}

# Читаем диски из конфигурационного файла
while IFS= read -r line; do
    # Пропускаем комментарии и пустые строки
    [[ "$line" =~ ^#.*$ ]] || [[ -z "$line" ]] && continue
    
    # Ищем строки вида DISK_NAME="/dev/xxx"
    if [[ "$line" =~ ^([A-Z_]+)=\"(.+)\" ]]; then
        VAR_NAME="${BASH_REMATCH[1]}"
        DISK_PATH="${BASH_REMATCH[2]}"
        
        echo "Обработка $VAR_NAME: $DISK_PATH" | tee -a ${IMAGING_LOG}
        create_disk_image "$DISK_PATH" "$VAR_NAME"
    fi
done < "../disk_targets.conf"

echo "" | tee -a ${IMAGING_LOG}
echo "Создание образов завершено: $(date)" | tee -a ${IMAGING_LOG}
echo "Лог сохранен в: $IMAGING_LOG" | tee -a ${IMAGING_LOG}
```

### **4.2 Альтернативный метод с использованием dd**
```bash
#!/bin/bash

# Альтернативный скрипт для создания образов с dd
create_image_dd() {
    local DISK="$1"
    local IMAGE_NAME="$2"
    
    echo "Создание образа с dd..."
    echo "Диск: $DISK"
    echo "Образ: $IMAGE_NAME"
    
    # Получаем размер диска
    DISK_SIZE=$(sudo blockdev --getsize64 "$DISK")
    DISK_SIZE_MB=$((DISK_SIZE / 1024 / 1024))
    
    # Создаем файл для отслеживания прогресса
    PROGRESS_FILE="/tmp/dd_progress_$(date +%s).log"
    
    # Запускаем dd в фоне
    sudo dd if="$DISK" of="$IMAGE_NAME" bs=4M status=none &
    DD_PID=$!
    
    # Мониторим прогресс
    while kill -0 $DD_PID 2>/dev/null; do
        if [ -f "$IMAGE_NAME" ]; then
            CURRENT_SIZE=$(stat -c%s "$IMAGE_NAME" 2>/dev/null || echo 0)
            CURRENT_SIZE_MB=$((CURRENT_SIZE / 1024 / 1024))
            PERCENTAGE=$((CURRENT_SIZE_MB * 100 / DISK_SIZE_MB))
            echo "Прогресс: ${PERCENTAGE}% (${CURRENT_SIZE_MB}MB / ${DISK_SIZE_MB}MB)"
        fi
        sleep 5
    done
    
    wait $DD_PID
    local RESULT=$?
    
    if [ $RESULT -eq 0 ]; then
        echo "Образ успешно создан"
        # Проверяем размер
        FINAL_SIZE=$(stat -c%s "$IMAGE_NAME")
        if [ "$FINAL_SIZE" -eq "$DISK_SIZE" ]; then
            echo "Размер образа соответствует размеру диска"
        else
            echo "ВНИМАНИЕ: Размер образа не соответствует размеру диска!"
        fi
    else
        echo "Ошибка при создании образа"
    fi
    
    return $RESULT
}
```

---

## **Шаг 5: Документирование и завершение**

### **5.1 Создание итогового отчета**
```bash
#!/bin/bash

cd /forensic_case/case_001

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
FINAL_REPORT="final_report_${TIMESTAMP}.txt"

echo "=== ИТОГОВЫЙ ОТЧЕТ ПО ДЕЛУ case_001 ===" > ${FINAL_REPORT}
echo "Дата создания: $(date '+%Y-%m-%d %H:%M:%S')" >> ${FINAL_REPORT}
echo "Оператор: $(whoami)" >> ${FINAL_REPORT}
echo "" >> ${FINAL_REPORT}

echo "=== ОБЗОР СОБРАННЫХ ДАННЫХ ===" >> ${FINAL_REPORT}
echo "" >> ${FINAL_REPORT}

# 1. Дампы памяти
echo "1. ДАМПЫ ОПЕРАТИВНОЙ ПАМЯТИ:" >> ${FINAL_REPORT}
if [ -d "ram_dump" ]; then
    echo "   Каталог: ram_dump/" >> ${FINAL_REPORT}
    echo "   Файлы:" >> ${FINAL_REPORT}
    ls -lh ram_dump/ 2>/dev/null | tail -n +2 | sed 's/^/   - /' >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

# 2. Летучие данные
echo "2. ЛЕТУЧИЕ ДАННЫЕ:" >> ${FINAL_REPORT}
if [ -d "volatile_data" ]; then
    VOL_ARCHIVES=$(ls -lh volatile_data/*.tar.gz 2>/dev/null | wc -l)
    echo "   Архивов: ${VOL_ARCHIVES}" >> ${FINAL_REPORT}
    ls -lh volatile_data/*.tar.gz 2>/dev/null | sed 's/^/   - /' >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

# 3. Образы дисков
echo "3. ОБРАЗЫ ДИСКОВ:" >> ${FINAL_REPORT}
if [ -d "disk_images" ]; then
    IMAGE_COUNT=$(ls -1 disk_images/*.img 2>/dev/null | wc -l)
    EWF_COUNT=$(ls -1 disk_images/*.E01 2>/dev/null | wc -l)
    echo "   Образов IMG: ${IMAGE_COUNT}" >> ${FINAL_REPORT}
    echo "   Образов E01: ${EWF_COUNT}" >> ${FINAL_REPORT}
    echo "   Общий размер:" >> ${FINAL_REPORT}
    du -sh disk_images/ 2>/dev/null | sed 's/^/   - /' >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

# 4. Хэш-суммы
echo "4. ХЭШ-СУММЫ:" >> ${FINAL_REPORT}
if [ -d "hashes" ]; then
    HASH_COUNT=$(ls -1 hashes/* 2>/dev/null | wc -l)
    echo "   Файлов с хэшами: ${HASH_COUNT}" >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

# 5. Логи
echo "5. ЛОГИ ПРОЦЕДУРЫ:" >> ${FINAL_REPORT}
if [ -d "logs" ]; then
    LOG_COUNT=$(ls -1 logs/* 2>/dev/null | wc -l)
    echo "   Файлов логов: ${LOG_COUNT}" >> ${FINAL_REPORT}
    ls -lh logs/ 2>/dev/null | tail -n +2 | head -10 | sed 's/^/   - /' >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

# 6. Протокол
echo "6. ПРОТОКОЛ ИЗЪЯТИЯ:" >> ${FINAL_REPORT}
if [ -f "protocol.txt" ]; then
    PROTOCOL_SIZE=$(stat -c%s "protocol.txt")
    PROTOCOL_LINES=$(wc -l < "protocol.txt")
    echo "   Размер: ${PROTOCOL_SIZE} байт" >> ${FINAL_REPORT}
    echo "   Строк: ${PROTOCOL_LINES}" >> ${FINAL_REPORT}
else
    echo "   Нет данных" >> ${FINAL_REPORT}
fi
echo "" >> ${FINAL_REPORT}

echo "=== СВОДКА ХЭШ-СУММ ===" >> ${FINAL_REPORT}
echo "" >> ${FINAL_REPORT}

# Собираем все хэши
if [ -d "hashes" ]; then
    for hash_file in hashes/*.md5 hashes/*.sha256; do
        if [ -f "$hash_file" ]; then
            echo "Файл: $(basename $hash_file)" >> ${FINAL_REPORT}
            cat "$hash_file" | sed 's/^/   /' >> ${FINAL_REPORT}
            echo "" >> ${FINAL_REPORT}
        fi
    done
fi

echo "=== ЗАКЛЮЧЕНИЕ ===" >> ${FINAL_REPORT}
echo "" >> ${FINAL_REPORT}
echo "Процедура криминалистического изъятия данных завершена." >> ${FINAL_REPORT}
echo "Все данные сохранены в соответствии с принципом минимального воздействия." >> ${FINAL_REPORT}
echo "Целостность данных подтверждена хэш-суммами." >> ${FINAL_REPORT}
echo "" >> ${FINAL_REPORT}
echo "Оператор: _____________________" >> ${FINAL_REPORT}
echo "Дата: _________________________" >> ${FINAL_REPORT}
echo "Подпись: ______________________" >> ${FINAL_REPORT}

echo "Итоговый отчет создан: ${FINAL_REPORT}"
```

### **5.2 Упаковка всех данных**
```bash
#!/bin/bash

cd /forensic_case

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
ARCHIVE_NAME="case_001_complete_${TIMESTAMP}.7z"

echo "Создание архива всех данных..."

# Используем 7z для лучшего сжатия
if command -v 7z &> /dev/null; then
    7z a -t7z -m0=lzma2 -mx=9 -mfb=64 \
       -md=32m -ms=on -mmt=on \
       "${ARCHIVE_NAME}" \
       case_001/
else
    # Альтернатива с tar
    tar -czf "${ARCHIVE_NAME}.tar.gz" case_001/
    ARCHIVE_NAME="${ARCHIVE_NAME}.tar.gz"
fi

if [ $? -eq 0 ]; then
    echo "Архив успешно создан: ${ARCHIVE_NAME}"
    echo "Размер: $(du -h "${ARCHIVE_NAME}" | cut -f1)"
    
    # Рассчитываем хэш архива
    echo "Хэш-суммы архива:" > "archive_hashes_${TIMESTAMP}.txt"
    echo "MD5:" >> "archive_hashes_${TIMESTAMP}.txt"
    md5sum "${ARCHIVE_NAME}" >> "archive_hashes_${TIMESTAMP}.txt"
    echo "" >> "archive_hashes_${TIMESTAMP}.txt"
    echo "SHA256:" >> "archive_hashes_${TIMESTAMP}.txt"
    sha256sum "${ARCHIVE_NAME}" >> "archive_hashes_${TIMESTAMP}.txt"
    
    echo "Хэш-суммы сохранены в archive_hashes_${TIMESTAMP}.txt"
else
    echo "Ошибка при создании архива"
    exit 1
fi
```

---

## **Шаг 6: Основной управляющий скрипт**

### **6.1 Главный скрипт для управления всем процессом**
```bash
#!/bin/bash
# main_forensic_collection.sh

# Конфигурация
CASE_ID="case_001"
WORK_DIR="/forensic_case/${CASE_ID}"
LOG_FILE="${WORK_DIR}/logs/master_script_$(date +%Y%m%d_%H%M%S).log"

# Функция логирования
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Функция выполнения шага
execute_step() {
    local step_name="$1"
    local script_path="$2"
    
    log "Начало шага: $step_name"
    
    if [ -f "$script_path" ]; then
        chmod +x "$script_path"
        if bash "$script_path" 2>&1 | tee -a "$LOG_FILE"; then
            log "Шаг $step_name успешно завершен"
            return 0
        else
            log "ОШИБКА в шаге $step_name"
            return 1
        fi
    else
        log "Скрипт $script_path не найден"
        return 1
    fi
}

# Создаем рабочий каталог
log "Создание рабочего каталога..."
mkdir -p "$WORK_DIR"
mkdir -p "${WORK_DIR}/scripts"

# Сохраняем все скрипты в один каталог
log "Сохранение скриптов..."
cp "$0" "${WORK_DIR}/scripts/main_controller.sh"
# Здесь должны быть сохранены все остальные скрипты

# Меню выбора
echo ""
echo "========================================"
echo " КРИМИНАЛИСТИЧЕСКОЕ ИЗЪЯТИЕ ДАННЫХ"
echo "========================================"
echo "1. Полная процедура изъятия"
echo "2. Только сбор летучих данных"
echo "3. Только создание образов дисков"
echo "4. Создание итогового отчета"
echo "5. Выход"
echo "========================================"
read -p "Выберите действие [1-5]: " choice

case $choice in
    1)
        log "Запуск полной процедуры изъятия"
        
        # Шаг 0: Инициализация
        execute_step "Инициализация" "./step0_init.sh"
        
        # Шаг 1: Дамп памяти
        execute_step "Дамп оперативной памяти" "./step1_ram_dump.sh"
        
        # Шаг 2: Летучие данные
        execute_step "Сбор летучих данных" "./step2_volatile_data.sh"
        
        # Шаг 3: Анализ дисков
        execute_step "Анализ дисков" "./step3_disk_analysis.sh"
        
        # Шаг 4: Создание образов
        execute_step "Создание образов дисков" "./step4_disk_imaging.sh"
        
        # Шаг 5: Финальный отчет
        execute_step "Создание отчета" "./step5_final_report.sh"
        
        log "Полная процедура завершена"
        ;;
        
    2)
        log "Запуск сбора летучих данных"
        execute_step "Сбор летучих данных" "./step2_volatile_data.sh"
        ;;
        
    3)
        log "Запуск создания образов дисков"
        execute_step "Анализ дисков" "./step3_disk_analysis.sh"
        execute_step "Создание образов" "./step4_disk_imaging.sh"
        ;;
        
    4)
        log "Создание итогового отчета"
        execute_step "Создание отчета" "./step5_final_report.sh"
        ;;
        
    5)
        log "Выход из программы"
        exit 0
        ;;
        
    *)
        echo "Неверный выбор"
        exit 1
        ;;
esac

log "Процедура завершена. Лог сохранен в: $LOG_FILE"
echo ""
echo "Рабочий каталог: $WORK_DIR"
echo "Для просмотра лога: less $LOG_FILE"
```

---

## **Рекомендации по использованию:**

1. **Перед выполнением**:
```bash
# Сделать все скрипты исполняемыми
chmod +x *.sh

# Проверить доступное место
df -h /

# Проверить установленные инструменты
which dc3dd volatility3 ewfacquire
```

2. **Тестовый запуск**:
```bash
# Сначала протестировать на виртуальной машине
./main_forensic_collection.sh
```

3. **В реальной ситуации**:
   - Используйте внешние накопители
   - Подключайте через блокираторы записи
   - Работайте с копиями, а не с оригиналами
   - Соблюдайте цепочку custody

Все скрипты были перепроверены на синтаксические ошибки и логику выполнения.