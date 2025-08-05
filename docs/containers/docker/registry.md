---
title: Работа с реестрами
---

# 🗃️ Работа с реестрами

!!! abstract "Что такое Docker Registry"

    **Docker Registry** — это централизованное stateless-хранилище Docker-образов.

    Оно позволяет:

    - **Хранить** версии контейнеров.
    - **Делиться** ими с командой или сообществом.
    - **Автоматизировать** CI/CD-процессы через `push` и `pull`.

Существует несколько типов реестров:

- **Публичные**.
    - *Docker Hub* — реестр по умолчанию.
    - *GitHub Container Registry* — интегрирован с GitHub.
- **Частные (On-Premise или в облаке)**.
    - *Harbor* — популярное Open Source решение от CNCF.
    - *GitLab Container Registry* — встроен в GitLab.
    - *JFrog Artifactory* — универсальный менеджер артефактов.
    - **Облачные**: Amazon ECR, Google GCR, Azure ACR.
- **Локальные**.
    - Образ `registry:2` — официальный от Docker для быстрой разработки и тестов.

---

## 🌐 OCI Distribution

!!! info "Современный стандарт"

    Docker Registry API (v2) больше не развивается как отдельный продукт.

    !!! info "Подробнее про **OCI Distribution** см. в источнике 👉 [**OCI Distribution**](https://github.com/opencontainers/distribution-spec)."

    Его заменяет спецификация **OCI Distribution**, поддерживаемая Docker, Podman, Harbor и всеми современными инструментами.
    
    Она предоставляет единый протокол для хранения не только образов, но и других артефактов (например, Helm-чарты, WASM-модули), а также поддержку цифровых подписей и расширенной безопасности.

| Характеристика | Docker Registry v2 | OCI Distribution |
| :--- | :--- | :--- |
| **Стандартизация** | Частично (проприетарный стандарт Docker Inc.) | ✅ **Да** (открытый стандарт OCI) |
| **Поддержка артефактов** | Только образы контейнеров | ✅ **Да** (образы, Helm-чарты, WASM, и др.) |
| **Актуальность** | ❌ Устаревший | ✅ **Современный** |
| **Поддержка Docker** | Частично (в режиме совместимости) | ✅ **Полная** |

!!! success "Ключевые возможности OCI Distribution"

    - **Хранение произвольных артефактов**. OCI-реестры могут хранить не только образы, но и Helm-чарты, WASM-модули, SBOM и другие типы данных. Это превращает реестр в универсальное хранилище артефактов.
    - **Стандарт для цифровых подписей**. Интеграция с `sigstore/cosign` для проверки подлинности и целостности артефактов.
    - **Манифесты и индексы**. Стандартизированные форматы для описания наборов образов (например, для разных архитектур `amd64`, `arm64`).
    - **Расширяемость**. Поддержка разных медиа-типов позволяет сообществу добавлять новые виды артефактов без изменения спецификации.

### 🐳 Поддержка Docker

- **Docker Registry v2**: Docker взаимодействует с ним через свой проприетарный API.
Хотя этот API лег в основу OCI, он не поддерживает многие современные возможности.
- **OCI Distribution**: Docker (начиная с версии 20.10) является нативным клиентом для OCI-совместимых реестров.
Он использует стандартные эндпоинты и медиа-типы OCI, что обеспечивает полную совместимость.

!!! example "Практический пример"

    Когда вы выполняете `docker pull`, современный клиент Docker сначала пытается использовать OCI-протокол.

    Если реестр его не поддерживает, он откатывается к старому API Docker Registry v2.

    Для пользователя это прозрачно, но под капотом используются разные механизмы.

### 📑 Манифесты и индексы

!!! abstract "Что это такое"

    - **Манифест (`manifest`)**: Это JSON-файл, описывающий один конкретный артефакт.
    Он содержит информацию о его конфигурации, слоях и их медиа-типах.
    - **Индекс (`index`)**: Это "манифест манифестов".
    Он представляет собой JSON-файл, который ссылается на несколько манифестов, обычно для разных платформ (например, `linux/amd64`, `linux/arm64`).

Когда вы выполняете `docker pull ubuntu`, Docker на самом деле скачивает индекс, определяет вашу архитектуру и затем загружает манифест и слои, соответствующие вашей платформе.

!!! example "Диаграмма OCI Pull Flow"

    ```txt
    +--------+                                           +--------------+
    | Client |                                           | OCI Registry |
    +--------+                                           +--------------+
        |                                                       |
        | 1. docker pull my-app:latest                          |
        |------------------------------------------------------>|
        |                                                       |
        | 2. Возвращает index.json                              |
        |<------------------------------------------------------|
        |                                                       |
        | 3. Выбирает манифест для linux/amd64                  |
        |------------------------------------------------------>|
        |                                                       |
        | 4. Возвращает manifest.json                           |
        |<------------------------------------------------------|
        |                                                       |
        | 5. Запрашивает слои (blobs)                           |
        |------------------------------------------------------>|
        |                                                       |
        | 6. Возвращает слои                                    |
        |<------------------------------------------------------|
    ```

### 📦 Расширяемость (ORAS)

OCI Distribution позволяет определять собственные типы артефактов (`artifactType`).

Это означает, что вы можете хранить в реестре практически что угодно, от Java-архивов до моделей машинного обучения.

!!! success "Пример: хранение JAR-файлов"

    1.  **Определить свой `artifactType`**: Например, `application/vnd.mycompany.jar.v1+java`.
    2.  **Использовать инструмент для загрузки**.

    ```bash
    # Загрузить JAR-файл в реестр
    oras push my-registry.com/my-app:1.0 \
      --artifact-type application/vnd.mycompany.jar.v1+java \
      ./my-app.jar:application/java-archive
    
    # Скачать JAR-файл из реестра
    oras pull my-registry.com/my-app:1.0
    ```

    Теперь ваш JAR-файл хранится в реестре как нативный артефакт, и его можно скачивать, версионировать и интегрировать в CI/CD, как и обычные Docker-образы.

!!! info "Подробнее про **ORAS** см. в источнике 👉 [**ORAS (OCI Registry as Storage)**](https://oras.land/)."

---

## 🧭 Именование образов

Docker использует стандартизированный формат для идентификации образов:
`<реестр>/<неймспейс>/<образ>:<тег>`.

!!! example "Примеры"

    1. **Официальный образ (Docker Hub)**.

          ```bash
          docker pull alpine
          # Эквивалентно:
          docker pull docker.io/library/alpine:latest
          ```

          `library` — это официальный неймспейс Docker Hub для проверенных образов.

    2. **Пользовательский образ (Docker Hub)**.

          ```bash
          docker pull myusername/myapp:1.0
          ```

          `myusername` — неймспейс, обычно совпадающий с логином.

    3. **Образ из приватного реестра**.

          ```bash
          docker pull registry.mycompany.com/project/backend:v2.5
          ```

          - `registry.mycompany.com` — адрес реестра.
          - `project` — неймспейс (проект, команда).
          - `backend` — имя образа.
          - `v2.5` — тег.

!!! warning "Проблема тега `:latest`"

    Тег `:latest` — это просто **указатель**, а не гарантия последней версии.
    Он может ссылаться на любой образ, который автор посчитал "последним".

    - **Непредсказуемость**. `latest` может измениться в любой момент, сломав сборку или продакшен.
    - **Плохая практика в CI/CD**. Всегда используйте конкретные теги (версии, хеш коммита) для воспроизводимости.

---

## 🔒 Безопасность

### 🛡️ Content Digest

У каждого образа есть уникальный идентификатор — **content digest** (SHA256-хеш).

Он вычисляется на основе содержимого слоев образа.

!!! success "Зачем это нужно"

    - **Гарантия неизменности**. Если хеш совпадает, вы используете тот же самый образ, байт в байт.
    - **Безопасность**. Теги можно переназначать, а хеш — нет.

!!! example "Использование digest"

    ```bash
    # Скачивание по digest
    docker pull alpine@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b
    ```

    В Kubernetes `imagePullPolicy: IfNotPresent` также опирается на digest для проверки наличия образа.

### ✍️  От DCT до Sigstore

!!! warning "Docker Content Trust (DCT) — Legacy"

    **Docker Content Trust (DCT)** и его инструмент **Notary v1** считаются устаревшими.

    Они были сложны в настройке и не получили широкого распространения.

!!! success "Современный стандарт: Sigstore и Cosign"

    Проект **Sigstore** (включая инструмент **Cosign**) стал индустриальным стандартом для подписи и верификации артефактов.

    Он проще в использовании и поддерживается крупными игроками.

    **Ключевые возможности**:

    - **Подпись артефактов**: `cosign sign`.
    - **Верификация**: `cosign verify`.
    - **Прозрачность**: Все подписи хранятся в публичном логе (rekor), что защищает от атак.

!!! example "Пример работы с Cosign"

    ```bash
    # 1. Установить Cosign
    # go install github.com/sigstore/cosign/cmd/cosign@latest

    # 2. Сгенерировать пару ключей
    cosign generate-key-pair
    # Будут созданы файлы cosign.key и cosign.pub

    # 3. Подписать образ приватным ключом
    cosign sign --key cosign.key my-registry.com/my-app:latest

    # 4. Проверить подпись публичным ключом
    cosign verify --key cosign.pub my-registry.com/my-app:latest
    ```

### 🧾 SBOM

**SBOM** — это "состав" вашего контейнера: список всех библиотек, их версий и лицензий.

Это критически важно для анализа уязвимостей.

- **Форматы**: SPDX, CycloneDX.
- **Инструменты**:
    - `syft`, `trivy`: быстрые и популярные сканеры.
    - `tern`: анализирует каждый слой образа для более глубокого исследования происхождения пакетов.

OCI-реестры позволяют хранить SBOM как артефакт, связанный с образом.

!!! example "Пример создания и привязки SBOM"

    ```bash
    # 1. Установить Syft
    # go install github.com/anchore/syft/cmd/syft@latest

    # 2. Сгенерировать SBOM для образа
    syft my-registry.com/my-app:latest -o spdx-json > sbom.spdx.json

    # 3. Привязать SBOM к образу в реестре с помощью Cosign
    cosign attach sbom --sbom sbom.spdx.json my-registry.com/my-app:latest
    ```

---

## 🚀 Локальный реестр

Для разработки и тестирования удобно использовать официальный образ реестра.

!!! warning "`registry:2` не поддерживает OCI Distribution"

    Стандартный образ `registry:2` не поддерживает OCI-артефакты (например, подписи `cosign` или SBOM).

    !!! info "Подробнее про **Harbor** см. в источнике 👉 [**Harbor**](https://goharbor.io/)."

### ▶️ Без аутентификации

```bash
docker run -d -p 5000:5000 --name registry --restart=always registry:2
```

!!! tip "Работа с локальным реестром"

    Чтобы отправить или скачать образ, нужно указать адрес и порт:

    ```bash
    # Тегирование
    docker tag myapp:latest localhost:5000/myapp
    # Отправка
    docker push localhost:5000/myapp
    # Скачивание
    docker pull localhost:5000/myapp
    ```

### 🔐 Настройка TLS

По умолчанию Docker требует TLS для всех реестров, кроме `localhost`.

Если вы хотите обращаться к локальному реестру с другой машины по IP, Docker откажет в доступе.

!!! danger "Решение: `insecure-registries`"

    Можно явно разрешить Docker использовать незащищенное HTTP-соединение.

    Отредактируйте файл `/etc/docker/daemon.json` (или создайте его):

    ```json
    {
      "insecure-registries": ["your-registry-ip:5000"]
    }
    ```

    После этого перезапустите Docker.

Для более безопасного сценария можно настроить TLS.

!!! success "Пошаговая инструкция"

    1.  **Сгенерировать сертификаты**.

        ```bash
        openssl req -newkey rsa:4096 -nodes -keyout domain.key \
          -x509 -days 365 -out domain.crt \
          -subj "/CN=your-registry-hostname"
        ```

        !!! warning "Важно"

            `CN` (Common Name) **должен** совпадать с хостнеймом, по которому вы будете обращаться к реестру.

            **Не используйте IP-адрес!** если DNS-имени нет, добавьте запись в `/etc/hosts`.

    2.  **Разместить сертификаты на клиенте**.

        На всех машинах, которые будут подключаться к реестру, создайте директорию и скопируйте туда сертификат:

        ```bash
        sudo mkdir -p /etc/docker/certs.d/your-registry-hostname:5000
        sudo cp domain.crt /etc/docker/certs.d/your-registry-hostname:5000/ca.crt
        ```

    3.  **Запустить реестр с TLS**.

        ```bash
        docker run -d -p 5000:5000 --name registry \
          -v "$(pwd)"/certs:/certs \
          -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
          -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
          registry:2
        ```

### ⚙️ Конфигурация

Для более сложных настроек (аутентификация, хранилище и т.д.) используется конфигурационный файл.

!!! example "Пример `config.yml` с базовой аутентификацией"

    ```yaml
    version: 0.1
    log:
      level: info
      formatter: text
    storage:
      filesystem:
        rootdirectory: /var/lib/registry
    http:
      addr: :5000
      headers:
        X-Content-Type-Options: [nosniff]
    auth:
      htpasswd:
        realm: basic-realm
        path: /auth/htpasswd
    ```

!!! success "Настройка аутентификации `htpasswd`"

    1.  **Создать файл с паролями**.

        ```bash
        # Установить утилиту (если требуется)
        # sudo apt-get install apache2-utils
        mkdir auth
        htpasswd -Bbn myuser mypassword > auth/htpasswd
        ```

    2.  **Запустить реестр с `config.yml` и `auth`**.

        ```bash
        docker run -d -p 5000:5000 --name registry \
          -v "$(pwd)"/config.yml:/etc/docker/registry/config.yml \
          -v "$(pwd)"/auth:/auth \
          registry:2
        ```

!!! tip "Альтернативные хранилища"

    Локальная файловая система — не единственный вариант.

    Реестр поддерживает облачные хранилища, например, Amazon S3:

    ```yaml
    storage:
      s3:
        bucket: my-docker-bucket
        region: us-west-1
        accesskey: ...
        secretkey: ...
    ```

### 🐞 Отладка и проверка

!!! info "Логирование"

    По умолчанию логи пишутся в `stdout`, поэтому их можно посмотреть стандартной командой:

    ```bash
    docker logs -f registry
    ```

!!! tip "Проверка через API"

    Реестр предоставляет HTTP API (v2) для взаимодействия.

    Это полезно для отладки.

    ```bash
    # Получить список всех образов (каталог)
    curl -X GET http://localhost:5000/v2/_catalog

    # Получить список тегов для конкретного образа
    curl -X GET http://localhost:5000/v2/myapp/tags/list
    ```

---

## 📦 Оптимизация образов

### 📏 Минимизация размера

Чем меньше образ, тем быстрее он скачивается и тем меньше у него поверхность для атаки.

!!! success "Лучшие практики"

    1.  **Используйте минимальные базовые образы**.

        - `scratch` — пустой образ, идеален для статически скомпилированных бинарников (Go, Rust).
        - `alpine` — легковесный дистрибутив Linux.
        - `distroless` (от Google) — содержит только рантайм языка, без пакетного менеджера и shell.

    2.  **Объединяйте команды `RUN`**. Каждый `RUN` создает новый слой.

        ```dockerfile
        # Плохо
        RUN apt-get update
        RUN apt-get install -y curl

        # Хорошо
        RUN apt-get update && apt-get install -y curl \
            && rm -rf /var/lib/apt/lists/*
        ```

    3.  **Используйте `multi-stage builds`**. Собирайте приложение в одном контейнере, а запускайте в другом, копируя только бинарник.

### 🚀 Сборщик BuildKit

!!! success "Что такое BuildKit"
    **BuildKit** — это современный сборщик образов, который заменил старый движок Docker. Он включен по умолчанию в свежих версиях Docker и предоставляет множество улучшений.

!!! tip "Ключевые возможности"
    - **Параллельная сборка**. BuildKit может одновременно собирать независимые стадии в `Dockerfile`, что значительно ускоряет процесс.
    - **Продвинутое кэширование**. Он умеет кэшировать не только слои, но и контекст выполнения команд, например, `apt-get update`.
    - **Опция `--squash`**. Позволяет "схлопнуть" все слои образа в один, уменьшая его итоговый размер. (Используйте с осторожностью, так как это может замедлить `pull` из-за потери кэша слоев).
    - **Секреты и SSH-агенты**. Позволяет безопасно передавать секреты и SSH-ключи во время сборки без их сохранения в слоях образа.

!!! example "Включение BuildKit (если отключен)"
    ```bash
    export DOCKER_BUILDKIT=1
    docker build .
    ```

!!! info "Подробнее про **BuildKit** см. в главе 👉 [**BuildKit**](./buildkit.md)."

### 📝 Метаданные: `LABEL`

Добавляйте метаданные для описания образа.

!!! info "Почему `org.opencontainers.image.*`"
    Это часть спецификации **Open Container Initiative (OCI)** и заменяет устаревший `LABEL maintainer=...`.
    
    Современные тулчейны (Kubernetes, сканеры безопасности, CI/CD) умеют читать эти метки автоматически для анализа, отображения в UI и применения политик.

!!! example "Примеры стандартных меток"
    ```dockerfile
    LABEL org.opencontainers.image.title="My Awesome App" \
          org.opencontainers.image.description="This is a backend service for my API." \
          org.opencontainers.image.version="1.2.0" \
          org.opencontainers.image.revision="a1b2c3d4" \
          org.opencontainers.image.source="https://github.com/my-org/my-app" \
          org.opencontainers.image.licenses="MIT" \
          org.opencontainers.image.authors="Your Name <you@example.com>"
    ```

---

## 🔄 CI/CD и автоматизация

Интеграция реестра с CI/CD — ключевая практика DevOps.

!!! abstract "Типичный пайплайн"

    1.  **Push в ветку**. Разработчик пушит код в Git.
    2.  **Запуск CI**. Система (GitHub Actions, GitLab CI) запускает сборку.
    3.  **Сборка и тегирование**.
        - Собирается Docker-образ.
        - Образ тегируется версией или хешем коммита (`myapp:1.2.5`, `myapp:a1b2c3d`).
    4.  **Push в реестр**. Образ загружается в корпоративный или публичный реестр.
    5.  **Деплой**. CD-система (ArgoCD, Flux) или скрипт скачивает новую версию образа из реестра и обновляет приложение в Kubernetes или на серверах.

!!! tip "Секреты в CI/CD"

    Для `docker login` в CI-пайплайнах используйте переменные окружения или встроенные хранилища секретов (GitHub Secrets, GitLab CI/CD Variables) для хранения `DOCKER_USERNAME` и `DOCKER_PASSWORD`.

---

## 💥 Типовые ошибки

!!! danger "x509: certificate signed by unknown authority"

    **Причина**: Клиент Docker не доверяет TLS-сертификату реестра.

    **Решение**: Добавьте корневой сертификат (CA) в системное хранилище или в директорию `/etc/docker/certs.d/<registry-hostname>:<port>/ca.crt`.

!!! danger "no basic auth credentials"

    **Причина**: Реестр требует аутентификацию, но клиент не предоставил учетные данные.

    **Решение**: Выполните `docker login <registry-hostname>` перед `pull` или `push`.

!!! danger "unauthorized: authentication required"

    **Причина**: Предоставленные учетные данные неверны или у пользователя нет прав на запрошенный ресурс.

    **Решение**: Проверьте логин/пароль и права доступа в настройках реестра.

!!! danger "blob unknown to registry"

    **Причина**: Один из слоев образа отсутствует в реестре. Часто возникает при прерванной загрузке (`push`).

    **Решение**: Повторно загрузите образ (`docker push`).

---

## 🛠️ Практические команды

### `docker login/logout`

Выполняют процедуру входа и выхода в Docker-реестр.

!!! success "Как это работает"

    Команда `docker login` сохраняет ваши учетные данные в файле `~/.docker/config.json` в зашифрованном виде.

!!! example "Пример"

    ```bash
    # Для Docker Hub
    docker login
    docker logout

    # Для приватного реестра
    docker login registry.mycompany.com
    docker logout registry.mycompany.com
    ```

### `docker search`

Ищет образы в Docker Hub.

!!! example "Пример"

    ```bash
    docker search ubuntu
    ```

### `docker tag`

Команда `docker tag` не создает новый образ, а лишь присваивает существующему локальному образу новый "ярлык" (алиас), который указывает, в какой реестр его отправлять.

!!! example "Пример"

    ```bash
    # 1. Собираем образ локально
    docker build -t myapp:local .

    # 2. Тегируем его для отправки в приватный реестр
    docker tag myapp:local registry.mycompany.com/project/myapp:1.2.0
    ```

### `docker push`

Отправляет тегированный образ в удаленный реестр.

```bash
docker push registry.mycompany.com/project/myapp:1.2.0
```

### `docker pull`

Загружает образ из реестра на локальную машину.

```bash
docker pull registry.mycompany.com/project/myapp:1.2.0
```
