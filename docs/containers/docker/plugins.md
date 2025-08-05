---
title: Плагины в Docker
---

# 🔌 Плагины в Docker

Плагины Docker — это мощный механизм расширения базовой функциональности Docker Engine без модификации его исходного кода.

Они позволяют интегрировать Docker с внешними системами для управления:

- томами хранения данных.
- сетевыми интерфейсами.
- системами логирования.
- механизмами авторизации.

!!! abstract "Тезис"

    Плагины превращают Docker из простого инструмента для запуска контейнеров в гибкую платформу, адаптируемую под конкретные инфраструктурные требования.

    Понимание их работы открывает возможности для глубокой интеграции Docker с существующими системами хранения, сетевыми решениями и инструментами безопасности.

---

## 🌐 Совместимость

Ключевые ограничения:

- Работают только с `dockerd` (основной демон Docker).
- Поддерживаются преимущественно в **нативных Linux-окружениях**.
- Ограниченная поддержка в Docker Desktop (macOS/Windows) и WSL2.
- Не работают с `containerd` напрямую.

---

## 📚 Типы плагинов

Docker поддерживает несколько типов плагинов, каждый из которых отвечает за свою область.

| Тип | Назначение | Популярные реализации |
| :--- | :--- | :--- |
| Volume | Управление томами хранения | NFS, S3, GlusterFS, облачные диски |
| Network | Организация сетевых соединений | Weave, Contiv, Calico |
| Logging | Обработка логов контейнеров | Fluentd, Splunk, ELK |
| Authorization | Контроль доступа к Docker API | `docker_auth`, Portainer |
| Credential | Управление учетными данными (gMSA) | Windows-специфичные решения |


!!! question "Что такое плагин?"

    Плагин - это HTTP-сервер, слушающий UNIX-сокет.

---

## 🧭 Архитектура

Взаимодействие Docker Engine с плагином происходит по принципу "клиент-сервер" через UNIX-сокет.

```txt
+-------------------+          HTTP over UNIX socket         +----------------------+
|   Docker Engine   | <------------------------------------> |    Docker Plugin     |
|   (dockerd)       |                                        | (Volume, Network...) |
+-------------------+                                        +----------------------+
         |                                                               |
         |          /run/docker/plugins/<plugin_name>.sock               |
         |                                                               |
         |<------------------------ OCI spec --------------------------->|
```

1.  Docker запускает плагин в изолированном контейнере.
2.  Плагин реализует HTTP-интерфейс, описанный в `config.json`.
3.  Все команды (Create, Mount, Remove и др.) приходят как POST-запросы.

---

## 🗺️ Карта API-маршрутов

Взаимодействие Docker Engine с плагином происходит по протоколу HTTP. Ниже представлена карта основных маршрутов для `VolumeDriver`:

| Действие | HTTP Метод | Маршрут |
| :--- | :--- | :--- |
| Активация плагина | `POST` | `/Plugin.Activate` |
| Создание тома | `POST` | `/VolumeDriver.Create` |
| Удаление тома | `POST` | `/VolumeDriver.Remove` |
| Монтирование тома | `POST` | `/VolumeDriver.Mount` |
| Получение пути | `POST` | `/VolumeDriver.Path` |
| Получение информации | `POST` | `/VolumeDriver.Get` |
| Список томов | `POST` | `/VolumeDriver.List` |

---

## 🎨 Создание плагина

Создание плагина — это разработка веб-сервера, который отвечает на HTTP-запросы от Docker Engine через UNIX-сокет.

Плагин должен быть совместим со спецификацией **OCI (Open Container Initiative)**.

-   **`config.json`**: Манифест, описывающий тип плагина, его интерфейс (сокет), точки входа и требуемые права (`capabilities`).
-   **`rootfs`**: Корневая файловая система, содержащая исполняемый файл плагина и все его зависимости.

```txt
/myplugin
  ├── config.json
  ├── rootfs/
  │   └── myplugin  # Скомпилированный бинарник
  └── plugin.json
```

!!! example "Полный пример на Go"

    Этот пример реализует основные методы интерфейса `VolumeDriver` и запускает HTTP-сервер.

    ```go
    package main

    import (
        "log"
        "sync"

        "github.com/docker/go-plugins-helpers/volume"
    )

    type myDriver struct {
        volumes map[string]string
        mutex   sync.Mutex
    }

    func (d *myDriver) Create(req *volume.CreateRequest) error {
        d.mutex.Lock()
        defer d.mutex.Unlock()
        d.volumes[req.Name] = req.Options["path"]
        return nil
    }

    // ... (другие методы VolumeDriver)

    func main() {
        driver := &myDriver{volumes: make(map[string]string)}
        h := volume.NewHandler(driver)
        log.Println(h.ServeUnix("myplugin", 0))
    }
    ```

!!! tip "Сборка rootfs"

    Собери бинарник и положи его внутрь `rootfs`:

    ```bash
    CGO_ENABLED=0 go build -o rootfs/myplugin main.go
    ```

    Используй `FROM scratch` или `alpine` как базу, если контейнеризировано.

!!! info "Формат ответа"

    При вызове `VolumeDriver.Create` плагин должен вернуть JSON-ответ.

    В случае успеха это пустой объект `{}`, а в случае ошибки — объект с полем `Err`:

    ```json
    // Успех
    {}

    // Ошибка
    {
      "Err": "A human-readable error message"
    }
    ```

---

## ⚠️ Ограничения

1.  **Устаревание API**.

    - API плагинов Docker не так активно развивается, как другие части экосистемы.
    Некоторые старые плагины могут быть несовместимы с новыми версиями Docker Engine (>24.0).
    - !!! info "Подробнее про **go-plugins-helpers** см. в источнике 👉 [**go-plugins-helpers**](https://github.com/docker/go-plugins-helpers)."

2.  **Альтернативы**.

    - Для Kubernetes предпочтительнее использовать **CSI (Container Storage Interface)** для томов и **CNI (Container Network Interface)** для сетей.
    - Плагины Docker остаются актуальными преимущественно для **standalone Docker** или в **edge-окружениях**, где нет оркестрации Kubernetes.

!!! note "Подробнее про Kubernetes"

    В Kubernetes плагины Docker устарели.

    Предпочтительны:

    - **CSI (Container Storage Interface)** — для управления томами.
    - **CNI (Container Network Interface)** — для сетей.

    Использование плагинов Docker оправдано в edge-сценариях, on-prem хостах без Kubernetes или в CI/CD-сборках, где нужен доступ к нестандартным хранилищам или сетям.

---

## 🔒 Безопасность

!!! danger "Плагины работают с высоким уровнем привилегий"

    Установка плагина эквивалентна предоставлению `root`-доступа к хост-системе.

    Плагин может:

    - Монтировать произвольные тома на хосте.
    - Запросить привилегии (capabilities), включая `SYS_ADMIN`.
    - Получить доступ к сокетам Docker или внешним API.

    Чтобы минимизировать риски:

    -   **Проверяйте разрешения**. перед установкой всегда анализируйте `capabilities` и `mounts` в выводе `docker plugin inspect`.
    -   **Политика "zero trust"**. относитесь к плагинам как к компонентам с максимальным уровнем доверия, равным самому Docker Daemon. Устанавливайте их только из абсолютно доверенных источников.

!!! tip "Продвинутая изоляция"

    Для повышения контроля можно запускать плагины в изолированных **user namespaces** или применять к ним строгие политики **SELinux/AppArmor**, чтобы ограничить их взаимодействие с хост-системой.

---

## 🐛 Отладка

Отладка плагинов может быть нетривиальной задачей из-за их взаимодействия с Docker Engine.

!!! question "Где находятся сокеты?"

    Docker Engine ищет сокеты плагинов в директории `/run/docker/plugins`.

!!! question "Как посмотреть логи плагина?"

    Плагины работают как легковесные контейнеры.

    Их логи можно посмотреть стандартными средствами:

    ```bash
    # 1. Найти ID контейнера, соответствующего плагину
    docker ps -a --filter "label=plugin"

    # 2. Посмотреть его логи
    docker logs <plugin_container_id>
    ```

    Также ошибки могут появляться в логах самого Docker Engine (например, через `journalctl -u docker.service` или в файле `/var/log/syslog`).

!!! question "Что делать, если плагин не запускается?"

    Проверьте логи.

    Частые причины: несовместимость версий API, нехватка прав (`capabilities`) в `config.json`, или ошибки в самом коде плагина.

---

## 🛠️ Практические команды

### `docker plugin install`

Устанавливает плагин из Docker Hub или другого репозитория.

При установке Docker запрашивает у пользователя разрешения, которые требуются плагину.

!!! example "Примеры"

    ```bash
    # Установка плагина для работы с томами на SSHFS
    docker plugin install vieux/sshfs

    # Установка с принудительным включением
    docker plugin install --grant-all-permissions vieux/sshfs
    ```

### `docker plugin ls`

Показывает список установленных плагинов, их версии и статус (включен/выключен).

!!! example "Пример вывода"

    ```bash
    ID                  NAME                DESCRIPTION           ENABLED
    f1b774d2a5a6        vieux/sshfs:latest  SSHFS volume driver   true
    ```

### `docker plugin enable/disable`

Включает или отключает плагин.

!!! warning "Важно"
    `docker plugin disable` требует предварительной остановки всех контейнеров, использующих данный плагин.

Отключенный плагин не может быть использован, но остается в системе.

!!! example "Примеры"

    ```bash
    docker plugin disable vieux/sshfs
    docker plugin enable vieux/sshfs
    ```

### `docker plugin rm`

Удаляет плагин из системы.

!!! info "Примечание"
    После выполнения `docker plugin rm` некоторые артефакты (например, данные конфигурации) могут остаться в директории `/var/lib/docker/plugins/`.

!!! example "Примеры"

    ```bash
    # Удалить один плагин
    docker plugin rm vieux/sshfs

    # Удалить с принуждением (если он активен)
    docker plugin rm -f vieux/sshfs
    ```

### `docker plugin inspect`

Показывает детальную информацию о плагине, включая его конфигурацию, требуемые разрешения и переменные окружения.

!!! example "Пример"

    ```bash
    docker plugin inspect vieux/sshfs
    ```

### `docker plugin create`

Создает плагин из локальной директории, содержащей `config.json` и `rootfs`.

!!! warning "Требования для `docker plugin create`"

    Команда `docker plugin create` требует **root-доступа** к Docker Engine и может быть недоступна в некоторых окружениях (например, Docker Desktop) без включения experimental-функций или специальной конфигурации.

!!! example "Пример"

    ```bash
    # Создать плагин из директории my-plugin
    sudo docker plugin create my-username/my-simple-plugin ./my-plugin
    ```

