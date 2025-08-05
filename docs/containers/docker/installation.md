---
title: Установка Docker
---

# 🐳 Установка Docker

!!! question "Что такое Docker"

    **Docker** — это стандарт де-факто для контейнеризации приложений, обеспечивающий изоляцию, переносимость и масштабируемость.

---

## 🐧 Установка на Linux

!!! warning "Установка через скрипт"

    Использовать официальный скрипт установки можно **только для разработки**.

    ```bash
    # Стабильная версия
    curl -fsSL https://get.docker.com | sudo sh

    # Экспериментальная версия
    curl -fsSL https://test.docker.com | sudo sh
    ```

!!! success "Рекомендация"
    
    Для установки Docker **рекомендуется** использовать пакетные менеджеры (`apt`, `dnf`).

!!! example "⚙️ Команды для `apt`"

    ```bash
    # Устанавливаем зависимости
    sudo apt update
    sudo apt install -y ca-certificates curl gnupg lsb-release

    # Добавляем официальный GPG-ключ Docker
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
      sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Добавляем репозиторий Docker
    echo \
      "deb [arch=$(dpkg --print-architecture) \
      signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Установка Docker Engine и компонентов
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

!!! example "⚙️ Команды для `dnf`"

    ```bash
    # Удаление старых версий
    sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest \
      docker-latest-logrotate docker-logrotate docker-engine

    # Установка зависимостей
    sudo dnf -y install dnf-plugins-core

    # Добавление официального репозитория Docker
    sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    # Установка Docker Engine и зависимостей
    sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # Запуск и включение автозапуска
    sudo systemctl enable --now docker
    ```

!!! info "🛡️ Работа с SELinux"

    В дистрибутивах на базе Red Hat SELinux включён по умолчанию.

    Docker умеет работать в этом окружении, но требует правильной настройки.

!!! example "⚙️ Настройка SELinux"

    Сначала проверяем уровень SELinux.

    ```bash
    getenforce
    ```

    Если `Enforcing` — значит работает полный контроль.

    Далее можно поступить двумя способами:

    - Перевести SELinux в разрешающий режим.
    - Настроить SELinux для работы с Docker.

    Для первого варианта просто выполняется команда ниже.

    ```bash
    sudo setenforce 0
    ```

    Во втором же случае сначал устанавливается `container-selinux`, а затем настраивается демон Docker.

    ```bash
    sudo dnf install -y container-selinux
    sudo nano /etc/docker/daemon.json

    # {
    #  "selinux-enabled": true
    # }
    ```

    И затем перезапустите Docker.

---

## 🍏 Установка на macOS

!!! abstract "💡 Варианты установки"

    На macOS вы можете использовать **Docker Desktop**, но всё больше разработчиков и DevOps-инженеров переходят на **Colima** как лёгкую, open-source альтернативу.

### 🐳 Docker Desktop

!!! question "Что входит в Docker Desktop"

    Docker Desktop включает: `docker`, `docker-compose`, `kubectl`, локальный Kubernetes и GUI-интерфейс управления.

```bash
brew install --cask docker
```

После установки:

1.  Запустите **Docker Desktop** из Applications.
2.  Подождите появления значка кита в трее.
3.  Готово.

### 🦙 Colima

!!! question "Что такое Colima"

    Colima использует *[Lima](https://github.com/lima-vm/lima)* и *[QEMU](https://www.qemu.org/)* для лёгкой виртуализации без закрытого кода.

Почему стоит выбрать Colima:

- Открытый исходный код.
- Ниже потребление памяти и CPU.
- Лучше работает на ARM (M1/M2).
- Нет навязчивой лицензии, как у Docker Desktop.

```bash
brew install colima docker docker-compose
colima start
```

Далее можно проверить установился ли Docker.

```bash
docker info
```

!!! tip "Совет"

    Для использования Kubernetes: `colima start --with-kubernetes`.

!!! info "Подробнее про **Colima** см. в главе 👉 [**Colima**](../../tools/colima.md)."

---

## 🪟 Установка на Windows

!!! info "💡 Зависимость от WSL"

    Docker Desktop требует установленного и активированного **WSL2**.

    Если он не установлен — Docker предложит всё настроить автоматически.

Чтобы установить Docker можно использовать утилиту `winget`:

```ps1
winget install --id Docker.DockerDesktop -e
```

После установки:

1.  Обязательно перезагрузите систему.
2.  Запустите Docker Desktop вручную.
3.  Проверьте, что работает WSL2.
