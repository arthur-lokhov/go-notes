---
title: Пакет flag
---
# 🏁 Пакет `flag`

Пакет `flag` предоставляет встроенные средства для разбора флагов командной строки — параметров, передаваемых при запуске программы.

---

## 📌 Описание

!!! info "Основная идея данного пакета"

    Cвязать переменные в коде с аргументами командной строки.

### 🔧 Как это работает

1.  **Определение флагов:** Через функции `flag.String()`, `flag.Int()`, `flag.Bool()` и т.д.
2.  **Парсинг:** В `main()` вызывается `flag.Parse()`, которая анализирует `os.Args[1:]`.
3.  **Использование:** После парсинга можно использовать значения флагов через переменные.

### 💬 Синтаксис флагов

Пакет `flag` поддерживает несколько стандартных форматов:

- `-flag` или `--flag` → `true` для `bool`
- `-flag=value` → установить значение
- `-flag value` → аналогично, но не для `bool`
- `--` → завершает разбор флагов

!!! note "Флаги не перезаписываются"

    Каждый вызов `-flag` добавляет значение (а не перезаписывает).

## 🧱 Структуры, интерфейсы и методы

### 🏷️ Структура `Flag`

!!! tip "Исходный код"

    https://go.googlesource.com/go/+/refs/heads/master/src/flag/flag.go;l=408

Структура `Flag` представляет текущие состояние одного флага.

```go
type Flag struct {
    Name     string
    Usage    string
    Value    Value
    DefValue string
}
```

| Поле       | Тип          | Назначение                                                                  |
| ---------- | ------------ | --------------------------------------------------------------------------- |
| `Name`     | `string`     | Имя флага, используется в командной строке (`-flag`)                        |
| `Usage`    | `string`     | Текст, выводимый при помощи (`flag.Usage` или `flag.PrintDefaults`)         |
| `Value`    | `flag.Value` | Интерфейс, хранящий текущее значение флага                                  |
| `DefValue` | `string`     | Значение по умолчанию в строковом виде (используется при генерации справки) |

### 🔄 Интерфейсы `Value` и `Getter`

!!! tip "Исходный код"

    https://go.googlesource.com/go/+/refs/heads/master/src/flag/flag.go;l=360

`Value` основной интерфейс для работы со значениями флагов.
Любой тип, который его реализует, может быть использован в качестве флага.

```go
type Value interface {
    String() string
    Set(string) error
}
```

Также есть интерфейс `Getter`, который расширяет `Value`, добавляя метод `Get()`.
Все стандартные типы флагов в пакете реализуют этот интерфейс.

```go
type Getter interface {
    Value
    Get() any
}
```

| Метод                                   | Подробности                                                                            |
| --------------------------------------- | -------------------------------------------------------------------------------------- |
| **Set(string)**                         | Вызывается каждый раз, когда флаг `-list` встречается в командной строке               |
| **String()**                            | Используется для вывода значения, в `flag.PrintDefaults()` и `fmt.Print`               |
| **Get()**                               | Позволяет получить "сырое" значение, чаще полезно при рефлексии или логике над флагами |

### 🗂️ Структура `FlagSet`

!!! tip "Исходный код"

    https://go.googlesource.com/go/+/refs/heads/master/src/flag/flag.go;l=389

`FlagSet` представляет собой набор определенных флагов.
Это позволяет создавать независимые наборы флагов, например, для реализации подкоманд (как в `git remote add ...`).

```go
type FlagSet struct {
	Usage func()
    // остальные скрытые внутренние поля
}
```

??? example "Использование `FlagSet`"

    ```go
    package main

    import (
        "flag"
        "fmt"
        "os"
    )

    func main() {
        // Создаём подкоманду "serve"
        serveCmd := flag.NewFlagSet("serve", flag.ExitOnError)
        port := serveCmd.Int("port", 8080, "port to serve on")
        verbose := serveCmd.Bool("verbose", false, "enable verbose logging")

        if len(os.Args) < 2 {
            fmt.Println("expected 'serve' subcommand")
            os.Exit(1)
        }

        switch os.Args[1] {
        case "serve":
            serveCmd.Parse(os.Args[2:])
            fmt.Printf("Serving on port %d (verbose: %v)\n", *port, *verbose)

            // Пример обхода установленных флагов:
            serveCmd.Visit(func(f *flag.Flag) {
                fmt.Printf("Установлен флаг: -%s = %s\n", f.Name, f.Value.String())
            })
        default:
            fmt.Println("Unknown command:", os.Args[1])
            os.Exit(1)
        }
    }
    ```

| Метод                          | Описание |
|--------------------------------|----------|
| `Parse(args []string)`         | Парсит аргументы командной строки |
| `Parsed() bool`                | Возвращает `true`, если парсинг уже был выполнен |
| `Set(name, value string)`      | Программно устанавливает значение для флага |
| `Lookup(name string)`          | Ищет флаг по имени и возвращает `*Flag` или `nil` |
| `Visit(fn func(*Flag))`        | Вызывает функцию для всех **установленных** флагов |
| `VisitAll(fn func(*Flag))`     | Вызывает функцию для **всех объявленных** флагов |
| `PrintDefaults()`              | Печатает usage-флаги и значения по умолчанию |
| `NFlag()`                      | Возвращает количество установленных флагов |
| `Arg(i int)`                   | Возвращает `i`-й позиционный аргумент |
| `Args()`                       | Возвращает слайс всех позиционных аргументов |
| `NArg()`                       | Количество позиционных аргументов |
| `Init(name, eh ErrorHandling)` | Переинициализирует `FlagSet` с новым именем и стратегией обработки ошибок |

!!! note "Поведение при ошибках (`ErrorHandling`)"

    При создании `FlagSet` можно указать, как он должен вести себя при ошибке парсинга. Для этого используются три константы:

    -   `ContinueOnError`: Вернуть ошибку, чтобы ее можно было обработать вручную.
    -   `ExitOnError`: Напечатать сообщение об ошибке и завершить программу с кодом 2 (поведение по умолчанию для `CommandLine`).
    -   `PanicOnError`: Вызвать панику с описанием ошибки.

    Пример:

    ```go
    fs := flag.NewFlagSet("build", flag.ContinueOnError)
    err := fs.Parse([]string{"-badflag"})
    if err != nil {
        fmt.Println("Ошибка парсинга:", err)
    }
    ```

---

## 🛠️ Функции для определения флагов

Для каждого основного типа есть две функции: `Xxx` и `XxxVar`.

-   `flag.Xxx(name, defaultValue, usage)`: Создает флаг, определяет для него переменную и возвращает указатель на нее.
-   `flag.XxxVar(p, name, defaultValue, usage)`: Связывает флаг с уже существующей переменной `p`.

| Тип         | Функция `Xxx`                               | Функция `XxxVar`                                 |
|-------------|---------------------------------------------|--------------------------------------------------|
| `bool`      | `p := flag.Bool(...)`                       | `var b bool; flag.BoolVar(&b, ...)`              |
| `int`       | `p := flag.Int(...)`                        | `var i int; flag.IntVar(&i, ...)`                |
| `int64`     | `p := flag.Int64(...)`                      | `var i64 int64; flag.Int64Var(&i64, ...)`         |
| `uint`      | `p := flag.Uint(...)`                       | `var u uint; flag.UintVar(&u, ...)`              |
| `uint64`    | `p := flag.Uint64(...)`                     | `var u64 uint64; flag.Uint64Var(&u64, ...)`       |
| `float64`   | `p := flag.Float64(...)`                    | `var f64 float64; flag.Float64Var(&f64, ...)`     |
| `string`    | `p := flag.String(...)`                     | `var s string; flag.StringVar(&s, ...)`          |
| `duration`  | `p := flag.Duration(...)`                   | `var d time.Duration; flag.DurationVar(&d, ...)` |

---

## 🧪 Пользовательские и функциональные флаги

| Функция                                               | Назначение                                                                           | Пример использования                                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `flag.Var(value Value, name, usage)`                  | Регистрирует флаг с **пользовательским типом**, реализующим интерфейс `flag.Value`   | `flag.Var(&myList, "list", "a list of values")`                                              |
| `flag.TextVar(p TextUnmarshaler, name, value, usage)` | Специализированный `Var` для типов, реализующих интерфейс `encoding.TextUnmarshaler` | `flag.TextVar(&ip, "ip", net.ParseIP("127.0.0.1"), "server IP")`                             |
| `flag.Func(name, usage, fn func(string) error)`       | Создает флаг, вызывающий **функцию обратного вызова** при установке значения         | `flag.Func("level", "log level", func(v string) error { log.SetLevel(v); return nil })`      |
| `flag.BoolFunc(name, usage, fn func(string) error)`   | Как `Func`, но работает как **логический флаг** (без значения)                       | `flag.BoolFunc("debug", "enable debug", func(v string) error { enableDebug(); return nil })` |

??? example "Примеры использования пользовательских флагов"

    Список строк с использованием `flag.Var`.

    ```go
    package main

    import (
        "flag"
        "strings"
    )

    type stringList []string

    func (s *stringList) Set(v string) error {
        *s = append(*s, v)
        return nil
    }

    func (s *stringList) String() string {
        return strings.Join(*s, ",")
    }


    func main() {
        var list stringList
        flag.Var(&list, "list", "a list of items")
    }
    ```

    IP-адрес с использованием `flag.TextVar`.

    ```go
    package main

    import (
        "flag"
        "net"
    )

    func main() {
        var ip net.IP
        flag.TextVar(&ip, "ip", net.ParseIP("127.0.0.1"), "server IP")
    }
    ```

    Установка уровня логирования с использованием `flag.Func`.

    ```go
    package main

    import (
        "flag"
        "fmt"
    )


    func main() {
        flag.Func("log-level", "set log level", func(s string) error {
            fmt.Println("уровень логирования:", s)
            return nil
        })
    }
    ```

    Активация режима отладки с использованием `flag.BoolFunc`.

    ```go
    package main

    import (
        "flag"
        "fmt"
    )


    func main() {
        flag.BoolFunc("debug", "включить отладку", func(_ string) error {
            fmt.Println("Режим отладки активирован")
            return nil
        })
    }
    ```
