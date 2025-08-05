---
title: BuildKit
---

# ⚙️ BuildKit

!!! question "Что такое BuildKit"

    **BuildKit** — это современный движок для сборки Docker-образов, разработанный командой **Moby Project** как замена устаревшего `legacy builder`, встроенного в Docker до версии 18.09.

    Он предоставляет:

    - высокую производительность,
    - улучшенное кэширование,
    - поддержку секретов,
    - безопасную работу с SSH,
    - мультиплатформенные сборки.

---

## 🚀 Почему BuildKit лучше

| Возможность | Legacy Builder | BuildKit |
| :--- | :--- | :--- |
| Параллельные стадии сборки | ❌ | ✅ |
| Гранулярное кэширование (`RUN`, `COPY`) | ❌ (только слой) | ✅ (пооперационное) |
| Мультиплатформенная сборка | ❌ | ✅ |
| Секреты и SSH во время сборки | ❌ | ✅ |
| Кэш-директории | ❌ | ✅ |
| Управление выводом (в `tar`, `oci`) | ❌ | ✅ |
| Отладка сборки (`progress`, `trace`) | ❌ | ✅ |
| Распределённые и удалённые builder'ы | ❌ | ✅ |

## 🧬 Архитектура BuildKit

!!! info "Принцип работы"

    BuildKit использует **DAG (ориентированный ациклический граф)** для оптимизации стадий сборки.
    Он анализирует зависимости между шагами Dockerfile и выполняет независимые шаги параллельно.

    Кэш реализуется на уровне каждого шага, а не слоя.
    Это означает, что даже одна изменённая команда `RUN` не затронет соседние, если они независимы.

    В основе BuildKit — **LLB (Low-Level Build)** — промежуточное представление Dockerfile.
    Это внутренний язык описания сборки, который BuildKit использует для создания графа зависимостей.

!!! example "⚙️ Пример работы BuildKit"

    ```txt
    [docker CLI / docker buildx]
               ↓  (gRPC API)
           [buildkitd]  ———— DAG & LLB ————
             ↓          (LLB execution & cache)
       [solver] ———→ [cache manager]
             ↓
       [frontend]  (парсит Dockerfile, генерирует LLB)
             ↓
       [workers]  (например, контейнерные рантаймы или rootless)
             ↓
       [executor] (выполнение команд: RUN, COPY и т.д.)
             ↓
       [snapshotter] (сохраняет слои и артефакты)
             ↓
          Файловая система / Образы
    ```

    1.  **docker CLI / docker buildx** — интерфейс пользователя для запуска сборки. Отправляет gRPC-запросы в buildkitd.
    2.  **buildkitd** — демон BuildKit, который управляет сборками.
    3.  **frontend** — парсер Dockerfile (или других форматов), который генерирует DAG в виде LLB (низкоуровневого языка сборки).
    4.  **solver** — основной компонент, который анализирует DAG, ищет оптимальный план сборки с учётом кэша.
    5.  **cache manager** — отвечает за хранение и повторное использование промежуточных артефактов.
    6.  **workers** — исполнительные единицы, которые запускают команды, могут работать в контейнерах или на хосте.
    7.  **executor** — запускает отдельные команды в рамках стадии сборки.
    8.  **snapshotter** — сохраняет результат каждого шага (слои) в кэш и файловую систему.

---

## 🛠️ Активация BuildKit

!!! tip "Способы активации"

    Чтобы использовать новый движок сборки:

    - **Для `docker build`**: `DOCKER_BUILDKIT=1`.
    - **Для `docker compose`**: `COMPOSE_DOCKER_CLI_BUILD=1`.

!!! success "Включить по умолчанию"

    BuildKit также можно включить через настройки Docker CLI.

    ```json
    {
      "features": {
        "buildkit": true
      }
    }
    ```

    Включите BuildKit по умолчанию в конфигурации Docker, чтобы все `docker build` автоматически использовали современный движок.

---

## 🧩 `buildctl`

!!! note "Альтернативный CLI"

    Хотя чаще всего BuildKit используется через `docker buildx`, его можно запустить как отдельный демон и управлять через gRPC API или `buildctl`.

!!! example "⚙️ Пример сборки напрямую через buildctl"

    ```bash
    buildctl build \
      --frontend=dockerfile.v0 \
      --local context=. \
      --local dockerfile=. \
      --output type=docker,name=myapp | docker load
    ```

---

## 🛠️ Docker Buildx

!!! question "Что такое `docker-buildx`"

    **`docker-buildx`** — это официальный CLI-плагин Docker, предоставляющий доступ ко всем возможностям сборочного движка **BuildKit**.

    Он заменяет устаревшую команду `docker build` и добавляет:

    - мультиплатформенную сборку,
    - расширенное кэширование,
    - безопасную работу с секретами,
    - сборку без необходимости запускать локальный демон Docker.

### 🤔 Отличия от `build`

| Характеристика | `DOCKER_BUILDKIT=1 docker build` | `docker buildx build` |
| :--- | :--- | :--- |
| **Интерфейс** | Стандартная команда Docker CLI | Плагин `buildx` (`docker build` на стероидах) |
| **Включение BuildKit** | Через переменную окружения | BuildKit всегда используется по умолчанию |
| **Мультиплатформенная сборка** | ❌ Не поддерживается | ✅ Поддерживается (через `--platform`) |
| **Кастомные билд-движки (builder instance)** | ❌ Только встроенный | ✅ Можно создавать и настраивать через `buildx create` |
| **Форматы вывода (exporter)** | Только Docker-образ | ✅ `docker`, `oci`, `local`, `tar`, `image` и др. |
| **Сетевые и SSH-фичи** (`--secret`, `--ssh`) | ✅ Частично | ✅ Полная поддержка |
| **Возможность пушить во время сборки** | ❌ Только `docker push` после сборки | ✅ Сборка и push в один шаг (`--push`) |
| **Доступность в CI/CD** | Ограничена | ✅ Удобна для headless/CI |
| **Управление кэшем (импорт/экспорт)** | ❌ Нет `--cache-to/from` | ✅ Полная поддержка |
| **Изоляция и масштабируемость** | Один глобальный BuildKit в Docker Engine | Несколько изолированных билд-инстансов (`builder`) |
| **Поддержка rootless-режима** | Ограниченно | ✅ Хорошо работает с rootless BuildKit |

!!! success "Используйте buildx"

    Использовать `docker buildx` вместо классического `docker build` — **стандарт на 2024+** в профессиональной разработке.

---

## 📛 Возможности BuildKit

### 🧬 Мульти-сборка

!!! info "Кросс-платформенная сборка"

    BuildKit позволяет собирать образы под разные архитектуры на одной машине — даже если сама машина на `x86_64`.

!!! example "⚙️ Пример: сборка образа под amd64 и arm64"

    ```bash
    docker buildx build \
      --platform linux/amd64,linux/arm64 \
      -t registry.io/my/image:latest \
      --push \
      .
    ```

!!! warning "Ограничение --load"

    Флаг `--load` не поддерживается с мультиархитектруной сборкой.

    Используйте `--push` или `--output`.

### 🔐 Работа с секретами

!!! info "Принцип работы секретов"

    Секрет монтируется как временный файл в `/run/secrets/<id>` внутри команды **RUN**.

    Он удаляется после выполнения и не сохраняется в слои образа.

!!! example "⚙️ Пример использования секрета"

    Dockerfile:

    ```dockerfile
    RUN --mount=type=secret,id=mysecret \
        cat /run/secrets/mysecret > /app/secret.txt
    ```

    CLI:

    ```bash
    docker buildx build \
      --secret id=mysecret,src=secret.txt \
      -t my-app \
      .
    ```

!!! danger "Не используйте ENV для секретов"

    1.  `type=secret` работает только внутри **RUN**.
    2.  Не передавайте секреты через **ENV** - они попадут в образ навсегда.

### 🔑 Работа с SSH

!!! example "⚙️ Пример работы с SSH"

    Dockerfile:

    ```dockerfile
    RUN --mount=type=ssh git clone git@github.com:private/repo.git
    ```

    CLI:

    ```bash
    docker buildx build \
        --ssh default \
        -t my-app \
        .
    ```

!!! tip "Настройка SSH-агента"
    Убедитесь, что SSH-агент запущен, и ключи добавлены.

    ```bash
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
    ```

!!! warning "Проблемы с QEMU"

    В сборках под чужую архитектуру (например, arm64) убедитесь, что настроен QEMU.

    Иначе `git clone` может не работать.

### 💾 Кэширование

!!! tip "Ускорение сборок"

    Кэш эффективно ускоряет повторные сборки, особенно для языков с dependency management (Go, Node.js, Python, Rust и т.д.).

!!! note "Ограничения --mount=type=cache"

    `type=cache` работает только внутри **RUN**.

    Он не влияет на **COPY** или **ADD**.


!!! example "⚙️ Пример кэширования зависимостей"

    ```dockerfile
    # Go-модули
    RUN --mount=type=cache,target=/go/pkg/mod \
        go mod download

    # npm
    RUN --mount=type=cache,target=/root/.npm \
        npm ci
    ```

!!! example "⚙️ Общий кэш между шагами"

    Для общего кэша используйте `id=`.

    ```dockerfile
    RUN --mount=type=cache,id=gomod,target=/go/pkg/mod go mod download
    RUN --mount=type=cache,id=gomod,target=/go/pkg/mod go build ./...
    ```

!!! example "⚙️ Настройка внешнего кэша (`--cache-to/--cache-from`)"

    ```bash
    docker buildx build \
      --cache-from=type=registry,ref=my-registry/cache:my-app \
      --cache-to=type=registry,ref=my-registry/cache:my-app,mode=max \
      -t my-app:latest \
      .
    ```

### 📦 Вывод сборки

| Тип вывода | Описание |
| :--- | :--- |
| `type=docker` | Загружает образ в Docker daemon (`--load`). |
| `type=image` | Сохраняет в BuildKit (в сочетании с `--push`). |
| `type=local` | Сохраняет артефакты сборки на хосте как обычные файлы. |
| `type=tar` | Экспортирует образ как `.tar`-архив. |
| `type=oci` | Экспортирует как OCI image layout. |

!!! example "⚙️ Сборка в tar-файл"

    ```bash
    docker buildx build \
      -o type=tar,dest=./app.tar \
      .
    ```

!!! example "⚙️ Вывод артефактов в директорию"

    ```bash
    docker buildx build \
      -o type=local,dest=./build-output \
      .
    ```

!!! tip "Когда использовать local и tar"

    Используйте:

    - `local`, когда нужны только бинарники или frontend-артефакты.
    - `tar`, когда сборка идет для оффлайн-установки или air-gapped-систем.

---

## 🐳 `docker-buildx`

### `build`

!!! abstract "🎯 Назначение"

    Основная команда для запуска сборки образа с использованием BuildKit.

!!! example "🎯 Популярные сценарии использования"

    **Локальная разработка и отладка**

    *Задача: быстро собрать образ для текущей архитектуры и загрузить его в локальный Docker для тестирования.*

    ```bash
    docker buildx build \
      --platform=local \
      -t my-app:dev \
      --load \
      .
    ```

    - `--platform=local`: Оптимизирует сборку для архитектуры вашего хоста.
    - `-t my-app:dev`: Присваивает удобный тег для локального использования.
    - `--load`: Загружает готовый образ прямо в локальный Docker daemon, делая его доступным для `docker run`.

    **CI/CD: Сборка и пуш в реестр**

    *Задача: собрать production-ready образ, используя внешний кэш, и отправить его в реестр.*

    ```bash
    docker buildx build \
      -t my-registry/my-app:latest \
      -t my-registry/my-app:v1.2.3 \
      --cache-from=type=registry,ref=my-registry/my-app:cache \
      --cache-to=type=registry,ref=my-registry/my-app:cache,mode=max \
      --push \
      .
    ```

    - `-t ...`: Присваивает несколько тегов (например, `latest` и версионный).
    - `--cache-from`: Использует кэш от предыдущих сборок, сохраненный в реестре.
    - `--cache-to`: Сохраняет кэш этой сборки обратно в реестр для будущих запусков.
    - `mode=max`: Включает все возможные слои в кэш, делая его максимально эффективным.
    - `--push`: Отправляет образ в реестр сразу после успешной сборки.

    **Мультиплатформенная сборка**

    *Задача: собрать образ для нескольких архитектур (amd64 и arm64) и отправить его в реестр с единым манифестом.*

    ```bash
    docker buildx build \
      --platform linux/amd64,linux/arm64 \
      -t my-registry/my-app:latest \
      --provenance=false \
      --push \
      .
    ```

    - `--platform linux/amd64,linux/arm64`: Указывает целевые архитектуры.
    - `--provenance=false`: Отключает SLSA/provenance аттестацию, если она не нужна (может вызывать проблемы с некоторыми реестрами).
    - `--push`: Обязателен, так как мультиплатформенные образы нельзя загрузить в локальный Docker daemon.

    **Сборка с использованием секретов и SSH**

    *Задача: собрать образ, которому нужен доступ к приватному Git-репозиторию и секретному токену.*

    ```bash
    docker buildx build \
      -t my-app:with-secrets \
      --secret id=npmrc,src=$HOME/.npmrc \
      --ssh default \
      --load \
      .
    ```

    - `--secret id=npmrc,src=...`: Безопасно монтирует файл секрета (например, `.npmrc`) в сборку. Он не сохранится в слоях образа.
    - `--ssh default`: Пробрасывает ваш стандартный SSH-агент в процесс сборки для доступа к приватным репозиториям.

    **Экспорт файловой системы**

    *Задача: получить результат сборки в виде локальной директории, а не Docker-образа.*

    ```bash
    docker buildx build \
      --output type=local,dest=./build-output \
      .
    ```

    - `--output type=local,dest=...`: Сохраняет артефакты сборки (например, скомпилированный бинарный файл) в указанную папку на хосте.

### `create`

!!! abstract "🎯 Назначение"

    Создает новый экземпляр builder'а.
    Builder — это изолированная среда для выполнения сборок.

```bash
# Создать новый builder с драйвером docker-container
docker buildx create --name mybuilder --driver docker-container

# Создать и сразу начать использовать новый builder
docker buildx create --name mybuilder --use
```

!!! tip "Драйвер docker-container"

    Использование драйвера `docker-container` необходимо для мультиплатформенных сборок, так как он запускает BuildKit в контейнере с поддержкой QEMU для эмуляции архитектур.

### `ls`

!!! abstract "🎯 Назначение"

    Показывает список всех доступных builder'ов.

```bash
# Показать все builder'ы и их статус
docker buildx ls
```

!!! note "Активный builder"

    Активный builder, который используется по умолчанию, будет отмечен звездочкой (`*`).

### `use`

!!! abstract "🎯 Назначение"

    Переключает активный builder.

```bash
# Сделать 'mybuilder' активным
docker buildx use mybuilder

# Вернуться к builder'у по умолчанию
docker buildx use default
```

### `inspect`

!!! abstract "🎯 Назначение"

    Выводит подробную информацию о builder'е.

```bash
# Посмотреть конфигурацию текущего builder'а
docker buildx inspect

# Посмотреть конфигурацию и запустить builder, если он не активен
docker buildx inspect --bootstrap mybuilder
```

### `stop`

!!! abstract "🎯 Назначение"

    Останавливает указанный экземпляр builder'а.

```bash
# Остановить builder с именем 'mybuilder'
docker buildx stop mybuilder
```

!!! note "Освобождение ресурсов"

    Остановка builder'а полезна для освобождения ресурсов, особенно если он работает в контейнере.

### `rm`

!!! abstract "🎯 Назначение"

    Удаляет один или несколько builder'ов.

```bash
# Удалить builder с именем 'mybuilder'
docker buildx rm mybuilder

# Удалить все неактивные builder'ы
docker buildx rm
```

!!! warning "Удаление кэша"

    Удаление builder'а также уничтожает его кэш.

### `prune`

!!! abstract "🎯 Назначение"

    Очищает кэш сборки.

```bash
# Удалить весь кэш сборки для текущего builder'а
docker buildx prune

# Удалить кэш, который не использовался более 7 дней (168 часов)
docker buildx prune --filter 'until=168h'
```

!!! success "Регулярная очистка"

    Регулярно очищайте кэш в CI/CD-системах, чтобы избежать переполнения диска.

### `du`

!!! abstract "🎯 Назначение"

    Показывает использование дискового пространства кэшем сборки.

```bash
# Показать размер кэша
docker buildx du
```

### `bake`

!!! abstract "🎯 Назначение"

    Выполняет сборку на основе определения из файла (например, `docker-compose.yml` или HCL).
    Это мощный инструмент для управления сложными сценариями сборки.

```bash
# Собрать цели, определенные в docker-compose.yml
docker buildx bake

# Собрать конкретную цель
docker buildx bake my-service

# Собрать из файла HCL
docker buildx bake --file=build.hcl
```

!!! example "⚙️ Пример использования HCL-файла"

    ```hcl
    target "myapp" {
      context = "./"
      dockerfile = "Dockerfile"
      tags = ["myapp:latest"]
      platforms = ["linux/amd64", "linux/arm64"]
    }
    ```

### `imagetools`

!!! abstract "🎯 Назначение"

    Предоставляет инструменты для работы с манифестами образов в реестре.
    Позволяет создавать мультиархитектурные манифесты и инспектировать образы без их полной загрузки.

```bash
# Создать манифест, объединяющий образы для разных архитектур
docker buildx imagetools create \
  -t myregistry/myapp:latest \
  myregistry/myapp:linux-amd64 \
  myregistry/myapp:linux-arm64

# Посмотреть информацию о манифесте в реестре
docker buildx imagetools inspect myregistry/myapp:latest
```

### `version`

!!! abstract "🎯 Назначение"

    Показывает версию плагина `buildx`.

```bash
docker buildx version
```