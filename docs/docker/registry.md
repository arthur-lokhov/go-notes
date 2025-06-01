# 🗃️ Реестры и пространства имен
<!--markdownlint-disable MD007 MD046-->

## 🌐 Что такое Docker Registry

**Docker Registry** — это централизованное хранилище Docker-образов, позволяющее:

- Хранить версии контейнеров;
- Делиться ими с другими разработчиками;
- Организовывать CI/CD через push/pull из реестра.


Существует несколько типов реестров:

- 🏭 **Публичные**:
    - [Docker Hub](https://hub.docker.com/) — по умолчанию.
    - [GitHub Container Registry](https://github.com/features/packages).
- 🏢 **Частные/корпоративные**:
    - [Harbor](https://goharbor.io/).
    - [GitLab Container Registry](https://docs.gitlab.com/user/packages/container_registry/).
    - [Amazon ECR](https://aws.amazon.com/ecr) и др.
- 🔒 **Локальные**:
    - Простой реестр `registry:2` (официальный образ от Docker).

---

## 🧭 Пространства имён (namespaces)

Docker использует следующую структуру идентификаторов образов: `<реестр>/<namespace>/<образ>:<тег>`.

### ✅ **Без имени** (официальные образы)

```bash
docker pull alpine
```

Эквивалентно:

```bash
docker pull docker.io/library/alpine:latest
```

> `library` — официальный namespace Docker Hub.

### 👤 С пользовательским namespace (Docker Hub)

```bash
docker pull myusername/myapp
```

Здесь `myusername` — это namespace (обычно логин на Docker Hub).

### 🏢 С приватным реестром

```bash
docker pull registry.mycompany.com/project/backend:1.0
```

Здесь:

- `registry.mycompany.com` — адрес Docker Registry.
- `project` — namespace.
- `backend` — имя образа.
- `1.0` — тег (если не указан, подставляется latest).

## 🔐 Аутентификация и docker login

Для большинства публичных и приватных реестров требуется авторизация:

```bash
docker login
```

Для логина в конкретный реестр:

```bash
docker login registry.mycompany.com
```

После этого можно push/pull образы, если есть доступ.

## 🛠️ Работа с образами: tag, push, pull

📌 `tag` — пометить локальный образ

```bash
docker tag myapp registry.mycompany.com/project/myapp:1.0
```

### 🔍 Что происходит:

- `myapp` — это исходный локальный образ (имя или ID), который уже существует у вас после сборки.
- `registry.mycompany.com/project/myapp:1.0` — новое имя (reference), по которому вы хотите этот образ сохранить/пушить.

> Это "метка" (тег + адрес) для Docker, чтобы он знал, куда отправлять образ при push.

📤 `push` — загрузить образ в реестр

```bash
docker push registry.mycompany.com/project/myapp:1.0
```

📥 `pull` — скачать образ из реестра

```bash
docker pull registry.mycompany.com/project/myapp:1.0
```

## 🚀 Пример локального реестра (registry:2)

Разворачиваем:

```bash
docker run -d -p 5000:5000 --name registry registry:2
```

Работа с ним:

```bash
docker tag myapp localhost:5000/myapp
docker push localhost:5000/myapp
docker pull localhost:5000/myapp
```

Для доступа по адресу `localhost:5000` к незащищённому реестру, может потребоваться разрешить небезопасные реестры в Docker конфиге (`/etc/docker/daemon.json`).

Пример:

```json
{
    // Другие опции...
    "insecure-registries": ["localhost:5000"]
}
```

## 🧾 Подготовка Dockerfile к публикации

### 🖋️ Использование LABEL

Инструкция `LABEL` добавляет метаданные в образ:

```Dockerfile
LABEL maintainer="you@example.com"
LABEL version="1.0"
LABEL description="Backend service for API"
```

### 🧼 Минимизация слоёв

Для компактности используйте объединённые инструкции:

```Dockerfile
RUN apt update && apt install -y curl \
    && rm -rf /var/lib/apt/lists/*
```

### ✅ Проверка на корректность

```bash
docker build -t myapp .
docker run --rm myapp
```

---

## 📘 Команды для работы с реестрами

Для удобства все команды собраны мною в одном файле: [Подробнее](./commands.md).
