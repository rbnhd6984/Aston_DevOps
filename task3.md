# Домашнее задание 3

1. Пишем скрипт отвечающий требованиям ДЗ

```bash
#!/bin/bash

# Переменные
APP_DIR="/opt/app"
LOG_FILE="$APP_DIR/log.txt"

# Условие с проверкой на отсутствие директории и файла
[ ! -d "$APP_DIR" ] && mkdir -p "$APP_DIR" && echo "Created folder /opt/app"
[ ! -f "$LOG_FILE" ] && touch "$LOG_FILE" && echo "Created file /opt/app/log.txt"

echo "App structure ready at $APP_DIR"
echo "Log file is recording..."

# Бесконечный цикл для генерации строки длиной до 20 символов
while true; do
    # Генерируем случайную длину от 1 до 20
    len=$(( RANDOM % 20 + 1 ))

    # Генерируем случайную строку из букв и цифр
    # Используем tr для фильтрации только допустимых символов
    random_str=$(tr -dc 'a-zA-Z0-9' < /dev/urandom 2> /dev/null | head -c "$len")

    # Записываем строку в лог-файл с новой строки
    echo "$random_str" >> "$LOG_FILE"

    # Ждём 17 секунд
    sleep 17
done
```
2. Помещаем скрипт в папку /opt/app и делаем его исполняемым

```bash
sudo cp ~/script.sh /opt/app
sudo chmod +x /opt/app/script.sh
```

3. Создаем сервис для реализации автозагрузки, пишем конфигурацию

```bash
sudo nano /etc/systemd/system/log-generator.service
```

```ini
[Unit]
# Даем описание сервису
Description=Random string log generator (every 17 seconds)

[Service]
# Задаем тип сервиса для постоянно работающих (не фоновых) процессов
Type=simple
# Путь к исполняемому скрипту
ExecStart=/opt/app/script.sh
# Параметры перезапуска
Restart=always
RestartSec=5
# Запуск от root для доступа к системной директории /opt/app
User=root
# Логирование stdout и stderr сервиса в журнал
StandardOutput=journal
StandardError=journal

[Install]
# Запуск при обычной (много-пользовательской) загрузке системы
WantedBy=multi-user.target
```
3. Создаем конфиг файл logrotate, пишем конфигурацию

```bash
sudo nano /etc/logrotate.d/log-generator
```

```conf
/opt/app/log.txt {
    daily              # ротировать ежедневно
    rotate 30          # хранить 30 архивов (месяц)
    compress           # сжимать старые логи (gzip)
    delaycompress      # не сжимать самый свежий архив (log.txt.1)
    missingok          # не падать, если файла нет
    notifempty         # не ротировать, если файл пустой
    create 644 root root  # создавать новый файл с правами после ротации
}
```