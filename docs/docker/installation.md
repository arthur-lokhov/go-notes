# 🐳 Установка Docker
<!--markdownlint-disable MD046-->

**Docker** позволяет запускать приложения в изолированных средах, обеспечивая повторяемость, масштабируемость и удобство разработки.

Этот документ охватывает установку Docker на:

- Linux (включая SELinux)
- macOS (через Homebrew)
- Windows (через WinGet)

## 📦 Установка Docker на Linux

Самый быстрый и официальный способ — через **автоматизированный скрипт**:

### 🔧 Установка стабильной версии

```bash
curl -fsSL https://get.docker.com | sh
```

### 🧪 Установка экспериментальной версии

!!! warning

    Не рекомендуется использовать экспериментальные версии в продакшене.

```bash
curl -fsSL https://test.docker.com | sh
```

После установки добавьте текущего пользователя в группу `docker`, чтобы использовать Docker без `sudo`:

```bash
sudo usermod -aG docker $USER
newgrp docker
sudo service docker restart
```

## 🔒 Поддержка SELinux на RHEL/Fedora/CentOS

На дистрибутивах RedHat-семейства включён **SELinux** — механизм контроля доступа на уровне ядра.
Docker умеет с ним работать, но для этого нужно немного подготовиться.

### 📌 Проверка SELinux

```bash
getenforce
```

Если вывод: `Enforcing`, SELinux работает в жёстком режиме.

Далее можно поступить двумя способами:

- Перевести SELinux в разрешающий режим.
- Настроить SELinux для работы с Docker.

#### Включаем разрещающий режим для SELinux

Для перевода SELinux в разрешающий режим выполняем команду:

```bash
sudo seten-force 0
```

#### Настройка SELinux для работы с Docker

Сначала установите пакет `container-selinux`:

```bash
sudo dnf install -y container-selinux
```

Далее убедитесь, что у Docker включена поддержка SELinux.
Отредактируйте `/etc/docker/daemon.json`.

```json
{
    // ... Другие опции
    "selinux-enabled": true
}
```

И перезагружаем демон Docker'а:

```bash
sudo systemctl restart docker
```

## 🍏 Установка Docker на macOS

Docker Desktop можно установить напрямую через `brew`:

```bash
brew install --cask docker
```

После установки:

- Откройте приложение **Docker** из папки **Applications**.
- Разрешите запуск, если macOS попросит.
- Дождитесь появления иконки кита в трее — Docker готов к использованию.

!!! info

    Docker Desktop использует встроенную виртуализацию (Apple HV или HyperKit) и поставляется с `docker`, `docker-compose`, `kubectl` и пр.

## 🪟 Установка Docker на Windows

На Windows 10/11 используйте `winget`:

```ps1
winget install --id Docker.DockerDesktop -e
```

После установки:

- Перезагрузите систему.
- Запустите Docker Desktop вручную (или через автозапуск).
- Дождитесь появления иконки кита в трее.

!!! info

    По умолчанию Docker Desktop использует WSL2. Если не установлен, Docker сам предложит его установить.

## ✅ Проверка установки Docker

Чтобы убедиться в правильности установки Docker, следует выполнить команду `docker version`.

```bash
docker version
```

Команда должна вывести, что-то похожее:

```bash
Client:
 Version:           28.1.1
 API version:       1.49
 Go version:        go1.23.8
 Git commit:        4eba377
 Built:             Fri Apr 18 09:49:45 2025
 OS/Arch:           darwin/arm64
 Context:           desktop-linux

Server: Docker Desktop 4.41.2 (191736)
 Engine:
  Version:          28.1.1
  API version:      1.49 (minimum version 1.24)
  Go version:       go1.23.8
  Git commit:       01f442b
  Built:            Fri Apr 18 09:52:08 2025
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.7.27
  GitCommit:        05044ec0a9a75232cad458027ca83437aae3f4da
 runc:
  Version:          1.2.5
  GitCommit:        v1.2.5-0-g59923ef
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Далее проверяем запуск контейнера через Docker:

```bash
docker run hello-world

# Hello from Docker!
# This message shows that your installation appears to be working correctly
```

---

> 🧭 Docker — это фундамент, на котором строятся современные инфраструктуры. С его помощью вы переходите от просто запуска к автоматизации и масштабированию.
