---
title: Git
---

# 🐙 Git

**Git** — это не просто набор команд, а распределённая система управления версиями, созданная для организации эффективного рабочего процесса.

Ключевая идея — рассматривать репозиторий как **единственный источник правды (Single Source of Truth)**.

!!! abstract "Source of Truth"

    Это концепция, согласно которой все изменения, код, конфигурация и история проекта хранятся в одном месте — репозитории.

    Это обеспечивает прозрачность, упрощает совместную работу и служит отправной точкой для автоматизации (CI/CD, GitOps).

## 🚀 Установка

!!! example "Установка Git-клиента"
    === "Linux (apt)"
        ```bash
        # Debian/Ubuntu
        sudo apt-get update
        sudo apt-get install -y git
        ```
    === "Linux (dnf)"
        ```bash
        # Fedora/CentOS
        sudo dnf install -y git
        ```
    === "Windows"
        ```bash
        # Установка через winget
        winget install -e --id Git.Git
        ```
        !!! info "Подробнее про **winget** см. в источнике 👉 [**официальном репозитории**](https://github.com/microsoft/winget-cli)."

!!! tip "💡 Самая новая версия"

    !!! warning "Пример для Debian/Ubuntu"

        Если версия в пакетном менеджере вас не устраивает, то можно собрать **Git** из исходников.

        ```bash
        sudo apt-get install -y install-info dh-autoreconf libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev asciidoc xmlto docbook2x
        sudo ln -s /usr/bin/db2x_docbook2texi /usr/bin/docbook2x-texi
        tar -zxf git-2.39.2.tar.gz
        cd git-2.39.2
        make configure
        ./configure --prefix=/usr
        make all doc info
        sudo make install install-doc install-html install-info
        ```

## ⚙️ Настройка Git

| Настройка | Описание | Рекомендуемое значение |
|:---|:---|:---|
| **user.name** | Имя пользователя для коммитов. | Ваше реальное имя. |
| **user.email** | Email пользователя для коммитов. | Корпоративная/основная почта. |
| **core.editor** | Редактор для коммитов/конфликтов. | `vim`, `code --wait`, `nano`. |
| **commit.template** | Шаблон сообщений коммита. | Путь к вашему шаблону. |
| **core.pager** | Пейджер для вывода. | `less`, `cat`, `nvimpager`. |
| **merge.tool** | Инструмент для слияния. | `vimdiff`, `meld`, `kdiff3`. |
| **diff.external** | Внешний инструмент сравнения. | `nvimdiff`, `icdiff`. |
| **init.defaultBranch** | Ветка по умолчанию. | `main` (индустриальный стандарт). |
| **pull.rebase** | Стратегия получения изменений. | `true` (для линейной истории). |
| **core.excludesfile** | Глобальный `.gitignore`. | Путь к вашему файлу. |
| **color.ui** | Цветной вывод. | `true` (для лучшей читаемости). |
| **credential.helper** | Хранение учетных данных. | `store` (для Linux), `osxkeychain` (Mac). |
| **gpg.program** | Путь к **GPG**. | Путь к вашему **GPG**. |
| **commit.gpgsign** | Подпись коммитов. | `true` (для production). |

!!! note "Уровни хранения"

    ```bash
    # Системные настройки (для всех пользователей)
    sudo git config --system <параметр> <значение>  # /etc/gitconfig

    # Глобальные настройки (для текущего пользователя)
    git config --global <параметр> <значение>       # ~/.gitconfig

    # Локальные настройки (для конкретного репозитория)
    git config --local <параметр> <значение>        # .git/config
    ```

    !!! danger "Не используйте `--system` без необходимости в multi-user средах."

!!! tip "💡 Просмотр настроек"

    ```bash
    # Показать все настройки с указанием их источника
    git config --list --show-origin

    # Проверить конкретную настройку
    git config user.email
    ```

!!! example "Моя конфигурация Git"

    ```ini
    [core]
        excludesfile = /Users/arthurlokhov/.gitignore_global
    [alias]
        lg = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
        contrib = shortlog -s -n --no-merges
        rbi = rebase --interactive --autosquash
        fixup = commit --fixup
        amend = commit --amend --no-edit
        save = "!f() { git stash push -u -m \"WIP: $(date +%Y-%m-%d)\"; }; f"
        undo = reset HEAD~1 --mixed
        review = diff --color-moved-ws=allow-indentation-change --ignore-space-change origin/main..HEAD
        think = diff --cached
        blame = blame -w -C -C -C
        wipe = "!f() { git reset --hard && git clean -fd; }; f"
        morning = "!f() { git lg fetch --all --prune && git lg --since='yesterday'; }; f"
    [user]
        # Укажите свои рабочие данные
        name = Артур Лохов
        email = arthurlokhov@gmail.com
    [commit]
        # Активировать GPG-подпись для всех коммитов.
        # Это гарантирует, что авторство коммита не может быть подделано.
        gpgsign = true
    [gpg]
        # Укажите путь к вашей GPG-программе, если она не находится в PATH.
        program = gpg
    [pull]
        # Всегда использовать rebase при pull.
        # Это позволяет поддерживать чистую, линейную историю коммитов и избегать лишних merge-коммитов на фиче-ветках.
        rebase = true
    [rebase]
        # Автоматически сжимать (squash) коммиты, помеченные как fixup/squash, при интерактивном rebase.
        # Это упрощает процесс исправления истории коммитов.
        autosquash = true
    [fetch]
        # Автоматически удалять локальные ветки, которые уже не существуют на удаленном репозитории.
        # Это помогает поддерживать рабочую область в чистоте.
        prune = true
    [color]
        # Включить цветной вывод Git.
        ui = true
    [credential]
        # Использовать системный менеджер учетных данных (если доступно) для безопасного хранения паролей.
        helper = osxkeychain
    ```

---

## 🌐 Файлы `.gitignore`

Файл `.gitignore` указывает **Git**, какие файлы или директории следует игнорировать в рабочем каталоге.

Игнорируемые файлы не отслеживаются, а значит, не попадают в коммиты.

Это нужно, чтобы в репозитории хранился только исходный код и важные артефакты, а сгенерированные файлы, зависимости, логи и персональные настройки редакторов оставались локально.

!!! info "🎯 Три уровня"

    Существует три основных уровня, на которых можно определить правила игнорирования:

    1.  **Локальный (`.git/info/exclude`):**
        *   **Назначение:** Правила, специфичные только для вашей локальной копии репозитория.
        *   **Пример:** Персональные скрипты, заметки, которые не должны попасть ни в `.gitignore` проекта, ни в глобальный файл.
        *   **Важно:** Этот файл не является частью репозитория и не передается другим.

    2.  **Проектный (`.gitignore` в корне репозитория):**
        *   **Назначение:** Правила, которые должны применяться для всех, кто работает с проектом. Это **стандартный** способ игнорирования.
        *   **Что здесь указывать:**
            *   Сгенерированные файлы: `*.log`, `build/`, `dist/`.
            *   Зависимости: `node_modules/`, `vendor/`.
            *   Файлы окружения с секретами: `.env`, `*.pem`.
        *   **Важно:** Этот файл является частью репозитория и должен быть закоммичен.

    3.  **Глобальный (`~/.gitignore_global`):**
        *   **Назначение:** Правила, которые применяются ко всем вашим репозиториям на локальной машине.
        *   **Что здесь указывать:**
            *   Файлы операционной системы: `.DS_Store`, `Thumbs.db`.
            *   Настройки **IDE**/редакторов: `.idea/`, `.vscode/`, `*.swp`.
            *   Директории с временными файлами.

!!! example "Создание и активация глобального `.gitignore`"

    ```bash
    # 1. Создаем файл (если его нет)
    touch ~/.gitignore_global

    # 2. Добавляем правила для ОС и редакторов
    echo -e ".DS_Store\nThumbs.db\n*.swp\n.idea/\n.vscode/" >> ~/.gitignore_global

    # 3. Указываем Git использовать этот файл
    git config --global core.excludesfile ~/.gitignore_global
    ```

!!! tip "💡 Полезные ресурсы"

    Существуют готовые шаблоны `.gitignore` для разных языков и фреймворков.

    !!! info "Подробнее про **шаблоны .gitignore** см. в источнике 👉 [**github/gitignore**](https://github.com/github/gitignore)."

---

## ✨ Git-алиасы

!!! abstract "💡 Философия хороших алиасов"

    Хорошие алиасы должны:

    1.  **Решать конкретные задачи** — быть специализированными инструментами.
    2.  **Быть безопасными** — минимизировать риск ошибок.
    3.  **Сохранять прозрачность** — не скрывать важные этапы работы.
    4.  **Интегрироваться в процесс** — дополнять ваш **workflow**.

### 📜 История и анализ

!!! example "Визуализированная история (`lg`)"
    
    Это улучшенная версия `git log` для анализа истории.
    
    ```bash
    git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
    ```

    **Сценарии использования:**

    *   **Перед слиянием веток:** `git lg feature-branch..main`.
    *   **Анализ временной шкалы:** `git lg --since='1 week ago'`.
    *   **Поиск изменений в коде:** `git lg -S'function_name'`.

!!! example "Статистика вклада (`contrib`)"

    Показывает количество коммитов по авторам, игнорируя merge-коммиты.

    ```bash
    git config --global alias.contrib "shortlog -sn --no-merges"
    ```

    **Пример использования:**

    *   Посмотреть вклад с начала года: `git contrib --since=2023-01-01`.

### 🛠️ Работа с коммитами

!!! example "Умное перебазирование (`rbi`)"

    Автоматизирует процесс `rebase --interactive --autosquash`.

    ```bash
    git config --global alias.rbi "rebase --interactive --autosquash"
    ```

    **Workflow:**

    1.  Создаем fixup-коммит: `git fixup abc1234` (см. алиас `fixup` ниже).
    2.  Запускаем интерактивный rebase: `git rbi main`.
    3.  **Git** автоматически применяет исправления.

!!! example "Быстрые исправления (`fixup` и `amend`)"

    === "Точечное исправление (`fixup`)"

        Создает коммит, который будет "встроен" в другой коммит во время интерактивного rebase.

        Используется вместе с `rbi`.

        ```bash
        git config --global alias.fixup "commit --fixup"
        ```

    === "Дополнение коммита (`amend`)"

        Добавляет изменения в последний коммит, не открывая редактор для изменения сообщения.

        !!! tip "Идеально для добавления забытых файлов или мелких исправлений кода без изменения смысла коммита."

        ```bash
        git config --global alias.amend "commit --amend --no-edit"
        ```

!!! example "Экстренные алиасы (`save`, `undo`)"

    === "Сохранить всё (`save`)"
        
        Быстро сохраняет все изменения (включая неотслеживаемые файлы) во временном хранилище (`stash`).

        ```bash
        git config --global alias.save "!f() { git stash push -u -m \"WIP: $(date +%Y-%m-%d)\"; }; f"
        ```

    === "Отменить коммит (`undo`)"

        Отменяет последний коммит, но оставляет изменения в рабочей директории.

        ```bash
        git config --global alias.undo "reset HEAD~1 --mixed"
        ```

### 🔍 Инспекция кода

!!! example "Превью изменений (`review`)"

    Улучшенная версия `git diff` для **Code Review**.

    ```bash
    git config --global alias.review "diff --color-moved-ws=allow-indentation-change --ignore-space-change origin/main..HEAD"
    ```

    **Ключевые флаги:**

    *   `--color-moved-ws=allow-indentation-change`: Улучшает определение перемещенного кода, игнорируя изменения в отступах.
    *   `--ignore-space-change`: Игнорирует изменения, связанные только с пробелами.

!!! example "Проверка перед коммитом (`think`)"

    Показывает изменения, которые готовы к коммиту (`git diff --cached`). Помогает ответить на вопрос: "Что я сейчас собираюсь закоммитить?".

    ```bash
    git config --global alias.think "diff --cached"
    ```

!!! example "Поиск «виновника» (`blame`)"

    Улучшенная версия `git blame`, которая помогает найти коммит, изменивший строку, даже если код был перемещен.

    ```bash
    git config --global alias.blame "blame -w -C -C -C"
    ```

    **Ключевые флаги:**

    *   `-w`: Игнорирует пробельные изменения.
    *   `-C -C -C`: Трижды используется для более глубокого поиска перемещенного или скопированного кода внутри того же коммита или из других коммитов.

### ⚠️ Опасные алиасы

!!! danger "Полная очистка (`wipe`)"

    Сбрасывает все изменения и удаляет все неотслеживаемые файлы.

    ```bash
    git config --global alias.wipe "!git reset --hard && git clean -fd"
    ```

### 🤝 Настройка алиасов

!!! success "Рекомендация"
    
    Для синхронизации алиасов внутри команды, создайте файл `.gitaliases` в корне репозитория.

    **.gitaliases:**

    ```ini
    [alias]
        lg = log --graph --abbrev-commit...
        review = diff --color-moved...
    ```

    Затем каждый член команды должен добавить его в свой глобальный конфиг:

    ```bash
    # Путь должен быть абсолютным
    git config --global include.path "/path/to/your/project/.gitaliases"
    ```

---

## 🌿 Стратегии ветвления

!!! note "Используйте **защищенные ветки** (**Protected Branches**) в **GitHub/GitLab**."

    -   **Требуйте Pull Request (PR).**
    -   **Обязательные проверки статуса (CI).**
    -   **Требуйте ревью.**
    -   **Требуйте подпись коммитов.**

### 🌳 Trunk-Based Development, GitFlow и другие

Существуют разные стратегии ветвления (`Trunk-Based`, `GitFlow`, `GitHub Flow`), выбор которых зависит от размера команды и типа проекта.

---

## 📝 Коммиты и версионирование

### 💬 Conventional Commits

Это спецификация для сообщений коммитов, которая делает их понятными для машин и людей (`<type>(<scope>): <description>`).

!!! example "Примеры Conventional Commits"
    ```
    feat(auth): добавить поддержку двухфакторной аутентификации
    fix(ui): исправить выравнивание кнопки на главной
    ```

### 🤖 Автоматизация сообщений и changelog

Используйте **Commitizen** для создания стандартизированных коммитов и **Husky** для управления Git-хуками.

## 🛸 Продвинутые темы и рабочие процессы

### 🚀 Git в CI/CD и Docker

-   **Триггеры в CI:** Пайплайны в **GitHub Actions** или **GitLab CI** запускаются на события `push`, `pull_request` или `tag`.

!!! example "Пример триггеров в GitHub Actions"
    ```yaml
    name: CI
    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - name: Run tests
            run: make test
    ```

-   **Проверка подписей в CI:** Для верификации коммитов используйте `git verify-commit`.
-   **Проблема с владельцем в Docker:** Чтобы исправить ошибку `detected dubious ownership`, выполните:
    ```bash
    git config --global --add safe.directory /path/to/your/repo
    ```

!!! example "Git в Dockerfile"
    Чтобы использовать метаданные **Git** (например, хеш коммита) при сборке, копируйте `.git` и исправляйте владельца.
    ```dockerfile
    FROM alpine
    WORKDIR /app
    # Устанавливаем Git
    RUN apk add --no-cache git
    # Копируем .git и исходный код
    COPY --chown=nobody:nogroup .git .git
    COPY --chown=nobody:nogroup . .
    # Устанавливаем безопасную директорию
    RUN git config --global --add safe.directory /app
    # Получаем хеш коммита
    RUN git rev-parse --short HEAD > /app/version.txt
    ```

### 🪝 Git хуки в CI/CD

-   **`pre-commit`**: Запускает линтеры и форматеры локально перед коммитом.
-   **`pre-push`**: Выполняет тесты перед отправкой изменений. Это последняя линия защиты перед тем, как код попадет на удаленный репозиторий.

!!! example "Пример pre-push хука"
    ```bash
    #!/bin/sh
    # Запускает тесты перед пушем
    echo "Running tests..."
    make test
    if [ $? -ne 0 ]; then
        echo "Tests failed. Push aborted."
        exit 1
    fi
    ```

### 🤖 GitOps: Git как источник правды

**GitOps** — это подход, при котором **Git** является единственным источником правды. **ArgoCD** и **FluxCD** синхронизируют состояние инфраструктуры с репозиторием.

### 🏢 Масштабная работа: Monorepos и LFS

-   **Git LFS (Large File Storage):** Используется для работы с большими бинарными файлами, обходя ограничения платформ (например, 100MB на **GitHub**).
-   **Инструменты для монорепозиториев:** **Nx**, **Lerna** и **Changesets** помогают управлять зависимостями и сборкой.

## 🛠️ Полезные инструменты и UI

-   **gh:** Официальный **CLI** от **GitHub**.
-   **tig:** Мощный текстовый интерфейс (**TUI**) для **Git**.
-   **lazygit:** Современный и удобный **TUI** для **Git**.
-   **gitk:** Простой графический интерфейс (**GUI**), поставляемый вместе с **Git**.

## 📚 Справочник команд Git

### 📋 Основные команды

| Команда | Описание |
|:---|:---|
| `git init` | Инициализирует новый репозиторий. |
| `git clone <repo>` | Клонирует репозиторий. |
| `git add <file>` | Добавляет файл в индекс. |
| `git commit` | Создает коммит. |
| `git status` | Показывает состояние файлов. |
| `git log` | Показывает историю коммитов. |
| `git diff` | Показывает изменения. |

### 🌿 Ветвление и слияние

| Команда | Описание |
|:---|:---|
| `git branch` | Управляет ветками. |
| `git checkout <branch>` | Переключается на другую ветку. |
| `git merge <branch>` | Сливает указанную ветку в текущую. |
| `git rebase <branch>` | Перемещает коммиты текущей ветки поверх другой. |

!!! danger "Опасность `git rebase` на публичных ветках"
    Не используйте `git rebase` на публичных (общих) ветках, так как это перезаписывает историю.

### 🤕 Восстановление и отладка

#### 🔍 Анализ истории

-   **`git blame <file>`**: Показывает, кто и когда изменял каждую строку файла.
-   **`git log -S"<string>"`**: Ищет коммиты по содержимому.

#### 🕵️‍♂️ Поиск проблемных коммитов

-   **`git bisect`**: Автоматический бинарный поиск для нахождения коммита, который вызвал ошибку.

!!! example "Использование git bisect"
    ```bash
    git bisect start
    git bisect bad          # Отметить текущий коммит как "плохой"
    git bisect good <hash>  # Отметить коммит без бага как "хороший"
    git bisect reset        # Завершить сессию
    ```

#### ⏪ Отмена изменений

-   `git reflog`: Ваш "журнал безопасности" для восстановления утерянных коммитов.
-   `git reset --hard HEAD@{1}`: Возвращает к предыдущему состоянию. **Осторожно: незакоммиченные изменения будут потеряны.**
-   `git restore <file>`: Отменяет изменения в файле.
-   `git restore --staged <file>`: Убирает файл из индекса.
