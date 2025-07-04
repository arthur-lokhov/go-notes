---
title: Пакеты
---
# 📦 Пакеты в Go

## 🧠 Концепция пакетов

Go использует модель модульной компоновки кода, где **вся программа состоит из пакетов** (_packages_), а пакет состоит из **одного или нескольких `.go`-файлов**, которые вместе определяют функции, типы, переменные и константы, доступные во всех файлах одного пакета.

---

## 📄 Порядок в `.go`-файле

Каждый `.go`-файл обязательно принадлежит какому-либо пакету и имеет строгую структуру:

1. `package` - определяет, к какому пакету принадлежит файл;
2. `import` - подключение нужных внешних пакетов;
3. Объявление: функции, переменные, типы и так далее.

```go
package mylib     // 1. Имя пакета

import "fmt"      // 2. Импорт

// 3. Код пакета
func SayHello() {
    fmt.Println("Hello!")
}
```

---

## 🚀 Функция `main()`

Программа на Go всегда запускается с пакета `main`, содержащего точку входа — функцию `main()`.

```go hl_lines="1 8"
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Println("My favourite number is", rand.Intn(10))
}
```

---

## ⚙️ Функция `init()`

Каждый пакет может содержать одну или несколько функций `init()`, которые вызываются автоматически **до** `main()`, но **после импорта всех зависимостей**.

То есть `init()` крайне удобно использовать для инициализации переменных и настройки.

```go hl_lines="1"
func init() {
	setting.AppVer = Version
	setting.ForgejoVersion = ForgejoVersion
	setting.AppBuiltWith = formatBuiltWith()
	setting.AppStartTime = time.Now().UTC()
}
```

Особенности `init()`:

- В одном файле может быть несколько `init()`, также как и в одном пакете;
- Порядок вызова `init()` зависит от порядка импортов и порядка определения в файле,
- `init()` **не может принимать аргументы** и **не возвращает значения**.

!!! danger "Используйте умеренно"
    Не злоупотребляйте `init()` - это ухудшает читаемость и управление зависимостями.
    Лучше использовать явные вызовы и конструкторы.

---

## 🏷️ Именование пакетов

При наименовании пакетов стоит предерживаться следующих правил:

- Имя пакета должно совпадать с последним сегментов его пути импорта:
`#!go import "math/rand"` -> `#!go package rand`;
- Имена пакетов пишутся в **нижнем регистреа, без подчеркиваний и аббревиатур с заглавными буквами** (`http`, а не `HTTP`);
- Имя пакета не может быть `_` (пустым индентификатором);
- Избегайте названий вроде `utils`, `common` - они не дают информации о предназначении;
- Go не поддерживает вложенные пакеты как в Java/Python, каждый каталог - это **один пакет**, независимо от пространства имен;
    ```go
    // foo/a.go
    package foo // ✅

    // foo/bar/b.go
    package foo // ❌ ошибка: имя пакета не соответствует каталогу

    // foo/bar/b.go
    package bar // ✅
    ```
- В одном каталоге **все `.go`-файлы должны принадлежать одному и тому же пакету**;
    ```go
    // foo/a.go
    package foo // ✅

    // foo/b.go
    package bar // ❌ ошибка: разные пакеты в одном каталоге
    ```
- Один `.go`-файл не может содержать **несколько `package` объявлений**.
    ```go
    package main
    package gotest // ❌ ошибка: ожидался сценарий, а найден 'package'
    ```

!!! tip "Хорошее имя пакета"

    💡 Хорошее имя пакета - это имя, которое читается как префикс для его экспортируемых элементов.
    Например, `bytes.Buffer` или `http.Client`.

---

## 🧭 Импорт пакетов

Импорт позволяет подключить другой пакет и использовать его **экспортируемые имена** (начинающиеся с заглавной буквы).

### 📤 Экспортируемые имена

Go применяет **простой и эффективный** механизм видимости:

- Имена с **заглавной буквы** - экспортируются и доступны вне пакета.
- Имена с **маленькой буквы** - видимы только внутри текущего пакета.

```go
fmt.Println(math.Pi) // ✅
fmt.Println(math.pi) // ❌ ошибка: pi не экспортировано
```

### ✅ Варианты импортов

| Синтаксис                   | Как обращаться к функции из пакета                     |
| --------------------------- | ------------------------------------------------------ |
| `import "math"`             | `math.Sin`                                             |
| `import m "math"`           | `m.Sin`                                                |
| `import . "math"`           | `Sin` (без префикса — не рекомендуется)                |
| `import _ "net/http/pprof"` | `init()` вызывается, но пакет не используется напрямую |

### 🚫 Циклический импорт

Go не позволяет пакету **импортировать самого себе напрямую или косвенно**.

```go hl_lines="4 13"
// a/a.go
package a

import "gotest/b" // ❗ Импортируем пакет b

func FromA() {
	b.FromB()
}

// b/b.go
package b

import "gotest/a" // ❗ Цикл: b импортирует a

func FromB() {
	a.FromA()
}
```

Это означает, что структура импорта должна быть **направленным графом без циклов**.
При проектировании архитектуры нужно избегать пересекающихся зависимостей.

---

## 📂 Группировка импортов

Рекомендуется использовать **факторизованный стиль** импорта:

```go
import (
    "fmt"
    "math"
)
```

??? info "Про стили импорта"

    В Go допустимо импортировать пакеты **множественно** или **факторизованно**.
    Рекомендуется использовать **факторизованный стиль**, как более читаемый и удобный при масштабировании.

    Факторизованный стиль - все импорты группируются внутри одной скобки `import (...)`:

    ```go
    import (
        "fmt"
        "math"
        "net/http"
    )
    ```

    Его предпочитают так как:

    - Удобно добавлять и удалять пакеты,
    - Можно группировать импорты по логике,
    - Это **де-факто стандарт**, который автоматически применяется командой `go fmt`.

    Множественны стиль (устаревший) - каждый импорт оформляется отдельной инструкцией `import "..."`:

    ```go
    import "fmt"
    import "math"
    import "net/http"
    ```

    Минусы такого подхода:

    - Быстро устаривает при росте количества импортов,
    - Хуже читается,
    - Не поддерживает группировку.

Go форматирует импортируемые пакеты по группам:

1. Стандартная библиотека.
2. Внешние зависимости.
3. Внутренние (локальные) модули.

??? info "Пример разделения пакетов по группам"

    ```go
    import (
        "context"
        "database/sql"
        "fmt"
        "os"
        "path"
        "path/filepath"
        "runtime"
        "strings"
        "sync/atomic"
        "testing"
        "time"

        "github.com/google/uuid"
        "github.com/stretchr/testify/assert"
        "github.com/stretchr/testify/require"
        "xorm.io/xorm/convert"

        "forgejo.org/models/db"
        "forgejo.org/models/unittest"
        "forgejo.org/modules/base"
        "forgejo.org/modules/git"
        "forgejo.org/modules/gitrepo"
        "forgejo.org/modules/graceful"
        "forgejo.org/modules/log"
        "forgejo.org/modules/optional"
        "forgejo.org/modules/process"
        "forgejo.org/modules/setting"
        "forgejo.org/modules/storage"
        "forgejo.org/modules/testlogger"
        "forgejo.org/modules/util"
        "forgejo.org/routers"
    )
    ```

---

## 📝 Комментарии к пакетам

Каждый пает должен начинаться с **комментария в стиле Godoc**, который кратко описывает его предназначение:

```go hl_lines="1"
// Package rand предоставляет функции для генерации псевдослучайных чисел.
package rand
```

Эти комментарии формируют документацию, которую можно просматривать через GoDoc.

!!! info "Что такое Godoc"

    Godoc - это инструмент документации для языка программирования Golang, который **автоматически генерирует** документацию на основе комментариев.

---

## 🧪 Тестовые пакеты

Go имеет встроенную поддержку тестирования через файлы с суффиксом `_test.go`.
Они могут принадлежать как тому же пакету, так и внешнему (с `_test`):

```go hl_lines="1 5"
package mylib_test // Тесты для пакеты mylib

import (
    "testing"
    "mylib" // Импорт пакета mylib
)

func TestThing(t *testing.T) {
    result := mylib.Thing()
    if result != "ok" {
        t.Errorf("expected ok, got %s", result)
    }
}
```

!!! tip "Использование суффикс `_test` в именовании пакета"

    Использование суффикса `_test` позволяет тестировать пакет как внешний клиент без доступа к внутренним методам - это усиливает модульность и API-дисциплину.

---

## 🔒 `internal` пакеты

Папка `internal/` позволяет **ограничить область использования пакета** - его можно импортировать только из родительского пакета и его подпакетов.

```
/project/
  internal/
    db/        ← доступен только внутри /project/...
  utils/        ← может быть импортирован кем угодно
```

В Go **может быть несколько `internal` директорий**, и каждая из них будет **ограничивать доступ на своем уровне**.

```
/project/
  internal/
    db/             ← доступен только внутри /project и его подпакетов
  utils/             ← может быть импортирован кем угодно
  db/
    internal/
      driver/        ← доступен только внутри /project/db и его подпакетов
```

Также можно использовать вложенность `internal`.

```
myproject/
  internal/
    db/
      internal/
        drivers/
          driver.go
      handler.go
  main.go
```

- ✅ `handler.go` может импортировать `internal/db/internal/drivers`.
- ❌ `main.go` — не может, так как он вне `internal/db`.

---

## 📦 Пакет `vendor`

!!! info "Историческая справка"

    `vendor` появился в Go 1.5 как экспериментальная функция, стал стандратом в 1.6.

    С появлением Go Modules в 1.11 его использование стало опциональным, но все ещё востребованным в определенных сценариях.

Пакет `vendor` в Go — это механизм для управления зависимостями проекта, позволяющий включать копии сторонних библиотек непосредственно в ваш проект.

Это обеспечивает:

- **Повторяемость сборок** - гарантия, что проект будет собираться с теми же версиями зависимостей;
- **Независимость от внешних репозиториев** - возможность сборки без доступа к интернету;
- **Контроль версий** - точная фиксация используемых версий библиотек.

### 🏗️ Структура `vendor`-директории

Типичная структура проекта с `vendor`:

```txt hl_lines="2"
myproject/
├── vendor/
│   ├── modules.txt
│   ├── github.com/
│   │   └── user/
│   │       └── repo/
│   │           ├── file1.go
│   │           └── file2.go
├── go.mod
├── go.sum
└── main.go
```

В директории `vendor` можно найти файл `modules.txt`. Данны файл содержит:

- Список всех вендорных пакетов;
- Информацию о замещениях (`replace`);
- Хэши для проверки целостности.

### ⚙️ Работа с `vendor`

Инициализация `vendor`:

```bash
go mod vendor
```

Эта команда:

1. Создает директорию `vendor/` если её нет.
2. Копирует все зависимости в `vendor/`.
3. Сохраняет точные версии из `go.mod`.

!!! tip "`vendor` только для конкретных пакетов"

    Можно вендорить только конкретные пакеты:

    ```bash
    go mod vendor github.com/user/repo
    ```

Сборка с использованием `vendor`:

```bash
go build -mod=vendor
```

!!! info "Флаг -mod=vendor указывает компилятору использовать зависимости из vendor вместо загрузки из сети."

    При использовании `-mod=vendor`:

    1. Компилятор ищет зависимости сначала в `vendor/`.
    2. Если не находит - ищет в кэше модулей (`$GOPATH/pkg/mod/`).
    3. И только после пытается скачать из сети.

Проверка целостности:

```bash
go mod verify
```

!!! info "Команда `mod verify` проверяет, чтобы все вендорные зависимости соответствовали `go.mod`."

### 🔄 Жизненный цикл `vendor`

1. Добавление зависимости
    ```bash
    go get github.com/user/repo@v1.2.3
    go mod vendor
    ```
2. Обновление зависимости
    ```bash
    go get github.com/user/repo@v1.2.4
    go mod vendor
    ```
3. Удаление зависимости
    ```bash
    go mod tidy
    go mod vendor
    ```

### ⚠️ Ограничения и подводные камни

1. Размер репозтория
    - `vendor` может значительно увеличить размер проекта.
    - **Решение**: использовать `.gitignore` для `vendor`.
2. Конфликты версий
    - Если разные пакеты требуют разные версии одной зависимости.
    - **Решение**: ручное разрешение конфликтов в `go.mod`.
3. Обновления
    - Легко забыть обновить вендорные зависимости после изменения `go.mod`.
    - **Решение**: автоматизировать в CI/CD.

---

## 🔗 Упоминание о линковщике и компиляции

Если вас интересует, как Go собирает и линкует пакетыв исполняемый файл, включая работу линковщика и этапы компиляции, обратитесь к главе ["Компиляция и сборка"](../sdk/build.md).

Там подробно разбирается:

- Как Go компилирует пакеты в объектные файлы (`*.a`);
- Роль линковщика в сборке финального бинарника;
- Как работает инструмент `go build` под капотом;
- Влияние кэша компиляции на скорость сборки;
- И многое другое.

