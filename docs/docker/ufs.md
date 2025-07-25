# 📁 Union File System (UFS)
<!--markdownlint-disable MD046-->

Чтобы понимать, как работают Docker-образы и контейнеры, нужно разобраться с UnionFS - файловой системой, которая применяется в Docker.

---

## 🔹 Что такое UFS

**Union File System (UFS)** — это способ **каскадного объединения** нескольких файловых систем (слоёв) в одну виртуальную.

Каждый слой может быть как `read-only`, так и `read-write`, но в рамках одного контейнера изменения допускаются **только в верхнем (RW) слое**.

В Docker UnionFS позволяет:

- Собирать образы из слоёв.
- Повторно использовать слои в разных образах.
- Эффективно хранить данные, избегая дублирования.

!!! info

    Виртуальная файловая система, созданная с помощью UnionFS, выглядит как обычная, но на деле — это стек из слоёв, наложенных друг на друга.

---

## 🔢 Слои образа

Каждый слой образа создаётся на основе предыдущего.

Пример:

1. `FROM ubuntu` → базовый слой (базовый образ).
2. `RUN apt update` → слой с изменённой системой (например, обновлённые пакеты).
3. `COPY . /app` → ещё один слой, добавляющий файлы проекта.

**Итог**: Три read-only слоя, объединённые в одну виртуальную файловую систему.
Образ — это и есть такая объединённая система, собранная из слоёв.

---

## 📦 Как работают контейнеры с UFS

Контейнер создаётся поверх образа, добавляя **read-write слой (RW Layer)**, который:

- Разрешает запись и изменения (в отличие от read-only слоёв образа).
- Является временным: исчезает при удалении контейнера.
- Позволяет "притворяться", что изменения затронули нижние слои, хотя на деле они только в RW-слое.

Если файл из read-only слоя изменяется — он **копируется в RW-слой (copy-on-write)**, и именно изменённая версия становится видимой контейнеру.

---

## ⚙️ Реализации UnionFS

Docker не реализует UnionFS сам по себе — он использует **одну из поддерживаемых реализаций** в зависимости от конфигурации хоста:

### 📊 Таблица реализаций UnionFS в Docker

| Драйвер                      | Поддержка ядра                     | Макс. кол-во слоёв                                   | Преимущества                                              | Недостатки                                                                            | Когда использовать                              |
| ---------------------------- | ---------------------------------- | ---------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **OverlayFS** / **overlay2** | Linux 3.18+ (overlay2: 4.0+)       | **128** слоёв (в overlay2)                           | Быстрый, простой, нативный для Linux                      | overlay (старый) не поддерживает директории с whiteout-файлами, overlay2 — исправляет | ✅ Рекомендуется по умолчанию                    |
| **AUFS**                     | Требует патчей ядра (вне mainline) | **127** слоёв                                        | Проверен временем, поддерживает глубокую иерархию         | Не входит в основное ядро, сложная настройка                                          | Только если необходима совместимость или legacy |
| **Btrfs**                    | Поддержка в ядре Linux             | **Нет явного лимита**, зависит от структуры          | Поддержка снапшотов, клонирования, сжатия                 | Требует Btrfs как основной ФС, может быть нестабильным под нагрузкой                  | Если нужна продвинутая работа с образами        |
| **ZFS**                      | Требует установки и настройки      | **Нет лимита на слои**, зависит от ZFS               | Надёжность, дедупликация, снапшоты, сжатие                | Сложность настройки, не во всех дистрибутивах доступен                                | Продвинутые случаи, требующие надёжности        |
| **devicemapper**             | Linux 2.6.9+                       | До **128** слоёв (зависит от конфигурации thin pool) | Поддерживает блочное управление, можно использовать с LVM | Медленный, сложный в управлении, требует тонкой настройки                             | Легаси или специфические окружения              |

### 🔎 Что важно помнить

1. **Overlay2** — наиболее производительная и рекомендуемая реализация для современных систем.
2. **AUFS** имеет жёсткое ограничение в 127 слоёв, что может стать проблемой при глубокой Dockerfile-инструкции.
3. **Btrfs/ZFS** дают максимум гибкости, но их стоит использовать, только если ты хорошо разбираешься в их конфигурации и умеешь их администрировать.
4. **devicemapper** может быть полезен в Red Hat-окружениях или если используется LVM-базовая инфраструктура.

### 🔍 Как узнать, какой драйвер используется

Выполни команду:

```bash
docker info
```

В выводе найди строку:

```bash
Storage Driver: overlay2
```

Это и есть текущая реализация UnionFS на твоей системе.

### 🔁 Как изменить драйвер

Чтобы изменить драйвер файловой системы, требуется сначала остановить Docker и очистить его данные:

```bash
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
```

А затем указать нужный драйвер в `/etc/docker/daemon.json`:

```json
{
    // ... Другие опции
    "storage-driver": "overlay2"
}
```

И перезапустить Docker.

```bash
sudo systemctl start docker
```

---

> Аббревиатура UFS во многих случаях используется для обозначения Unix File System.
> Файловую систему с каскодно-объединенным монтированием чаще обозначают как UnionFS.
