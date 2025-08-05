---
title: Docker Compose
---

# 🎼 Docker Compose

!!! abstract "Что такое docker-compose.yml"

    **`docker-compose.yml`** — это YAML-файл конфигурации для **Docker Compose**, утилиты, предназначенной для определения и запуска многоконтейнерных Docker-приложений.

    В этом файле вы описываете все компоненты вашего приложения (сервисы, сети, тома) и их взаимосвязи в декларативном формате. Compose читает этот файл и одной командой (`docker compose up`) создает и запускает всю вашу среду.

    Современный Compose, соответствующий *Compose Specification*, является стандартом де-факто и используется не только Docker, но и другими инструментами.

---

## 🏛️ `docker-compose.yml`

Файл имеет четкую иерархическую структуру.

На верхнем уровне находятся ключи, определяющие глобальные компоненты вашего стека.

```yaml
# Версия спецификации (информационная, не влияет на поведение)
version: '3.9'

# Основной раздел, где описываются все контейнеры (сервисы)
services:
  ...

# Раздел для определения именованных томов (volumes)
volumes:
  ...

# Раздел для определения кастомных сетей
networks:
  ...

# Раздел для управления конфигурационными файлами (только для Swarm)
configs:
  ...

# Раздел для управления секретами (только для Swarm)
secrets: 
  ...
```

!!! tip "YAML-схема для IDE"

    Для улучшения опыта разработки (DX) и получения автодополнения в вашей IDE, вы можете подключить официальную **JSON-схему** *Compose Specification*.

    **Пример для VS Code (`.vscode/settings.json`):**

    ```json
    {
      "yaml.schemas": {
        "https://raw.githubusercontent.com/compose-spec/compose-spec/master/schema/compose-spec.json": "docker-compose*.yml"
      }
    }
    ```

!!! warning "Только для Docker Swarm"

    Разделы `configs` и `secrets` на верхнем уровне `docker-compose.yml` работают **только в режиме Docker Swarm**.

    При использовании с обычным `docker compose` они будут проигнорированы, что может привести к ошибкам в приложении.

### ⚙️ Параметры сервисов

Это сердце `docker-compose.yml`.

Каждый ключ в разделе `services` представляет собой отдельный контейнер (сервис) вашего приложения.

```yaml
services:
  web:
    # --- Базовая конфигурация ---
    image: nginx:1.25-alpine  # Имя и тег Docker-образа для использования
    build:  # Параметры для сборки образа из Dockerfile
      context: ./web  # Путь к директории с Dockerfile и исходным кодом
      dockerfile: Dockerfile  # Имя Dockerfile (если отличается от стандартного)
      args: { APP_ENV: production }  # Аргументы, доступные только во время сборки
      target: builder  # Конкретная стадия multi-stage сборки
      ssh: default # Проброс SSH-ключей агента для доступа к приватным репозиториям
      cache_from:  # Источники кеша для ускорения сборки
        - type=registry,ref=my-registry/app:buildcache
        - type=local,src=./.build_cache
    container_name: my-nginx  # Имя контейнера (не рекомендуется для масштабируемых сервисов)
    hostname: webhost  # Имя хоста внутри контейнера
    profiles: ["frontend"]  # Профили, позволяющие запускать только часть сервисов
    command: ["nginx", "-g", "daemon off;"]  # Переопределяет команду по умолчанию (CMD) в образе
    entrypoint: ["/docker-entrypoint.sh"]  # Переопределяет точку входа (ENTRYPOINT) в образе

    # --- Безопасность и ресурсы ---
    read_only: true  # Запускает файловую систему контейнера в режиме "только для чтения"
    cap_drop: [ALL]  # Отключает все capabilities ядра Linux для повышения безопасности
    security_opt: [no-new-privileges:true]  # Запрещает процессу получать новые привилегии
    user: "1001:1001"  # UID:GID пользователя, от которого запускается процесс в контейнере
    ulimits:  # Лимиты на системные ресурсы
      nofile: { soft: 65536, hard: 65536 }  # Максимальное количество открытых файлов
    sysctls:  # Настройки параметров ядра (sysctl) внутри контейнера
      net.core.somaxconn: 1024

    # --- Переменные окружения ---
    environment:  # Устанавливает переменные окружения
      - NODE_ENV=production
      - "CACHE_SIZE=${CACHE_SIZE:-1024}" # Установка значения по умолчанию
    env_file: ./.env  # Загружает переменные окружения из файла

    # --- Сеть ---
    ports: ["80:80"]  # Проброс портов в формате ХОСТ:КОНТЕЙНЕР
    expose: ["8080"]  # Открывает порт для доступа из других сервисов в той же сети
    networks: [frontend]  # Подключает контейнер к указанным сетям
    dns: [8.8.8.8]  # Устанавливает кастомные DNS-серверы
    extra_hosts:  # Добавляет записи в /etc/hosts контейнера
      - "my-db.local:192.168.1.100"
    network_mode: "host" # ⚠️ Использовать сетевой стек хоста (только для Linux)

    # --- Файловая система ---
    volumes: ["./code:/var/www/html", "db_data:/var/lib/postgresql/data"]  # Монтирование томов и **bind mounts**
    tmpfs: /tmp  # Монтирование временной файловой системы в RAM
    working_dir: /var/www/html  # Рабочая директория внутри контейнера

    # --- Жизненный цикл и зависимости ---
    restart: always  # Политика перезапуска контейнера
    stop_grace_period: 30s # Время ожидания корректного завершения процесса перед SIGKILL
    depends_on:  # Определяет зависимости между сервисами
      db: { condition: service_healthy }  # Ждет, пока сервис 'db' не станет здоровым
    healthcheck:  # Проверка состояния работоспособности сервиса
      test: ["CMD", "curl", "-f", "http://localhost"]  # Команда для проверки
      interval: 30s  # Интервал между проверками
      timeout: 10s  # Таймаут на выполнение проверки
      retries: 3  # Количество попыток перед тем, как считать сервис нездоровым
      start_period: 40s  # Время на запуск контейнера до начала проверок
    
    # --- Дополнительные параметры ---
    init: true # Добавляет init-процесс для корректной обработки сигналов
    stdin_open: true # Держать STDIN открытым (эквивалент `docker run -i`)
    tty: true # Выделить псевдо-TTY (эквивалент `docker run -t`)
    privileged: true # ⚠️ Дает контейнеру расширенные права доступа к хосту
    shm_size: "256m" # Размер разделяемой памяти (/dev/shm)

    # --- Метаданные и логирование ---
    labels:  # Метаданные для контейнера
      com.example.description: "Web service"
    logging:  # Настройки логирования
      driver: "json-file"  # Драйвер логирования
      options:  # Опции драйвера
        max-size: "10m"

    # --- Развертывание (только для Docker Swarm) ---
    deploy:  # Параметры для развертывания в режиме Swarm
      replicas: 3  # Количество реплик сервиса
      resources:  # Ограничения на ресурсы
        limits: { cpus: "0.5", memory: 512M }
```

### 💾 Общие тома

Этот раздел позволяет определять именованные тома, которыми могут совместно пользоваться несколько сервисов. Это предпочтительный способ для хранения персистентных данных.

```yaml
volumes:
  db_data: # Имя тома, которое используется в сервисах
    driver: local # Драйвер тома (local, nfs, и т.д.)
    driver_opts: # Опции для драйвера
      type: none
      device: /home/user/data # Путь на хосте для bind mount
      o: bind
    labels: # Метаданные для тома
      purpose: "Database persistent storage"
```

### 🌐 Сети

Позволяет создавать кастомные сети для изоляции сервисов. Сервисы видят друг друга по именам только в том случае, если они находятся в одной сети.

```yaml
networks:
  frontend:
    driver: bridge # Стандартная сеть для одного хоста
  backend:
    driver: overlay # Сеть для Docker Swarm, работает на нескольких хостах
    attachable: true # Разрешает ручное подключение контейнеров
    ipam: # Управление IP-адресацией
      driver: default
      config:
        - subnet: 172.16.238.0/24 # Задать подсеть
```

---

## 🚀 Продвинутые практики

### 🏗️ Несколько файлов

Для разных окружений (разработка, тестирование, продакшн) принято использовать несколько `docker-compose` файлов.

Docker Compose автоматически объединяет их в одну конфигурацию.

-   **`docker-compose.yml`**: Базовый файл, содержащий общие для всех окружений сервисы и настройки.
-   **`docker-compose.override.yml`**: Файл для локальной разработки.
**Compose загружает его автоматически**, если он лежит рядом с основным файлом.
В нем обычно переопределяют `command` для **hot-reload**, пробрасывают порты и монтируют исходный код в контейнер.
-   **`docker-compose.prod.yml` / `docker-compose.staging.yml`**: Файлы для конкретных окружений.
В них задают политики `restart`, переменные окружения для продакшена, настройки логирования и т.д.

!!! example "Пример структуры проекта"

    ```
    project/
    ├── docker-compose.yml          # База
    ├── docker-compose.override.yml # Для разработки (авто-подгрузка)
    ├── docker-compose.prod.yml     # Для продакшена
    ├── .env                        # Переменные для локальной разработки
    ├── .env.prod                   # Переменные для продакшена
    └── web/
        └── Dockerfile
    ```

    Для разработки (файлы `docker-compose.yml` и `docker-compose.override.yml` подхватятся автоматически):

    ```bash
    docker compose up -d
    ```

    Для продакшена (явно указываем файлы):

    ```bash
    docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
    ```

!!! note "Порядок объединения"

    Compose читает файлы по порядку и переопределяет параметры.

    Например, `ports` из `override` заменят `ports` из базового файла.


### 📄 Переменные окружения

Docker Compose автоматически ищет файл `.env` в директории проекта и загружает из него переменные окружения.

Эти переменные можно использовать внутри `docker-compose.yml` для подстановки.

!!! example "Пример использования `.env`"

    `.env`:

    ```env
    TAG=latest
    WEB_PORT=8080
    ```

    **Пример `docker-compose.yml`:**

    ```yaml
    services:
      web:
        image: myapp:${TAG:-dev} # Если TAG не задан, используется 'dev'
        ports:
          - "${WEB_PORT}:80"
    ```

### 📄 Экспорт конфигурации

Чтобы увидеть итоговую, объединенную конфигурацию, используйте команду:

```bash
# Для разработки
docker compose config

# Для продакшена
docker compose -f docker-compose.yml -f docker-compose.prod.yml config
```

### 🧩 Композиция

Позволяет переиспользовать конфигурацию сервиса из другого файла.

**Внимание:** эта возможность считается устаревшей в пользу использования нескольких файлов (`-f` флаг).

```yaml
# common.yml
services:
  base: { image: alpine }

# docker-compose.yml
services:
  web:
    extends: { file: common.yml, service: base }
```

### 🧪 Тестирование

Можно создать отдельную стадию в `Dockerfile` для тестов и запускать ее через `build.target`.

```dockerfile
# Dockerfile
FROM node:20 AS test
WORKDIR /app
COPY . .
RUN npm ci && npm test
```

```yaml
# docker-compose.test.yml
services:
  sut:
    build: { context: ., target: test }
```

### 🧬 Встраивание

Если в нескольких сервисах используются одни и те же параметры (например, переменные окружения, volume, labels и др.), вместо дублирования можно использовать YAML-ключ `<<`, чтобы встроить общую часть из отдельного блока (alias).

!!! example "Пример базового шаблона"

    ```yaml
    x-common-env: &common-env
      environment:
        - NODE_ENV=production
        - LOG_LEVEL=info

    x-common-volumes: &common-volumes
      volumes:
        - logs:/var/log/app

    services:
      service-a:
        image: app-a:latest
        <<: [*common-env, *common-volumes]

      service-b:
        image: app-b:latest
        <<: [*common-env, *common-volumes]

    volumes:
      logs:
    ```

!!! tip "Объединение с конкретными параметрами"

    Можно переопределить параметры, встроенные через `<<`, в теле сервиса:

    ```yaml
    services:
      web:
        image: app:latest
        <<: *common-env
        environment:
          - NODE_ENV=development  # переопределяет значение из якоря
    ```

!!! warning "Не все YAML-парсеры одинаковы"

    Хотя `<<` — часть **YAML-спецификации**, некоторые редакторы и инструменты могут не поддерживать её корректно.

    Убедитесь, что ваш инструмент (например, `docker compose`) обрабатывает **merge key** корректно — Docker Compose на базе *Compose Specification* поддерживает.

---

## ⚠️ Подводные камни

!!! danger "⛓️ `depends_on` — Не дожидается готовности сервиса"

    **Проблема:** `depends_on` управляет только **порядком запуска** контейнеров, а не их полной готовностью к работе.

    Ваш сервис может запуститься раньше, чем его зависимость (например, база данных) будет готова принимать соединения.
    
    **Решение:**

    1.  **Использовать `healthcheck`:** Добавьте проверку состояния в зависимый сервис.
    2.  **Использовать `condition: service_healthy`:** Укажите в `depends_on`, что нужно дождаться успешного `healthcheck`.

    ```yaml
    services:
      db:
        image: postgres
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres"]
          interval: 5s
          timeout: 3s
          retries: 5
      app:
        image: myapp
        depends_on:
          db:
            condition: service_healthy
    ```
    
    Для более надежного контроля готовности в сложных сценариях рекомендуется использовать специализированные скрипты, такие как `wait-for-it.sh` или `dockerize`.

!!! danger "configs и secrets — Только для Docker Swarm"

    **Проблема:** Секции `configs` и `secrets` верхнего уровня работают **только в режиме Docker Swarm** (`docker stack deploy`), а не с `docker compose up`.

    Их использование в обычном режиме приведет к ошибке.

    **Решение для `docker compose`:**

    -   **Конфиги:** Используйте `volumes` для монтирования файлов конфигурации.

        ```yaml
        services:
          app:
            volumes:
              - ./myapp.conf:/etc/myapp.conf:ro
        ```

    -   **Секреты:** Используйте переменные окружения, загружаемые из `.env` файла.

        ```yaml
        services:
          app:
            environment:
              - DB_PASSWORD=${DB_PASSWORD}
        ```

    **Никогда не хардкодьте секреты!**

    Храните их в `.env` файлах (добавленных в `.gitignore`) или используйте системы управления секретами вашего CI/CD.

!!! danger "container_name — Потенциальные конфликты"

    **Проблема:** Жестко заданное имя контейнера (`container_name`) мешает масштабированию (`docker compose up --scale app=3`) и может вызывать конфликты имен, особенно в CI/CD средах.

    **Решение:** Не используйте `container_name`, если для этого нет веской причины.

    Docker Compose автоматически генерирует уникальные имена вида `<project>_<service>_<index>`, что предотвращает конфликты.

!!! danger "restart: always — Может зациклить сбои"

    **Проблема:** Политика `restart: always` будет перезапускать контейнер бесконечно, даже если он падает сразу после старта из-за ошибки в конфигурации.
    Это может привести к "спаму" в логах и излишней нагрузке на систему.

    **Решение:** Используйте `restart: on-failure` (перезапускать только при ненулевом коде выхода) или `restart: unless-stopped` (перезапускать всегда, кроме случая, когда контейнер остановлен вручную).

!!! danger "Docker Compose ≠ Оркестратор"

    **Проблема:** Docker Compose — это инструмент для разработки и простого развертывания на одном хосте.

    Он не является полноценной системой оркестрации и не предоставляет таких функций, как балансировка нагрузки между узлами, самовосстановление (self-healing) или автоматическое масштабирование в кластере.

    **Решение:** Для производственных сред используйте полноценные оркестраторы:

    -   **Docker Swarm:** Простой и встроенный в Docker.
    -   **Kubernetes:** Индустриальный стандарт (можно использовать `kompose` для конвертации).
    -   **Nomad, OpenShift, Rancher** и другие.
