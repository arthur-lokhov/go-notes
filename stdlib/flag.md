# 🏁 Пакет flag: работа с флагами командной строки

Представь, что ты написал утилиту, которая делает что-то полезное. Но как пользователь должен указать, что именно делать? Где хранить конфигурацию? Как включить отладочный режим?

Для этого и нужен пакет `flag` — он превращает аргументы командной строки в понятные переменные твоей программы.

В отличие от сложных библиотек вроде `cobra` или `kingpin`, `flag` — это минимализм. Никаких подкоманд из коробки, никаких красивых help-экранов с цветами. Зато он встроен в стандартную библиотеку, прост как молоток и работает везде.

---

## 🧠 Интуитивное понимание

Пакет `flag` работает по принципу «объяви → распарси → используй».

```
┌─────────────────────────────────────────────────────────────┐
│  1. Объявляешь флаги                                        │
│     flag.String("host", "localhost", "server host")         │
│     ↓                                                       │
│  2. Вызываешь Parse()                                       │
│     flag.Parse()                                            │
│     ↓                                                       │
│  3. Используешь значения                                    │
│     fmt.Println(*host)                                      │
└─────────────────────────────────────────────────────────────┘
```

Это не магия — это просто договорённость. Ты говоришь пакету: «Я ожидаю вот такие флаги». Он смотрит в `os.Args`, находит знакомые имена и заполняет переменные.

---

## 📦 Базовое использование

### Простой пример

Начнём с самого простого — программы, которая принимает имя пользователя и выводит приветствие:

```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    // Объявляем флаг
    name := flag.String("name", "World", "Имя для приветствия")
    
    // Парсим аргументы
    flag.Parse()
    
    // Используем значение
    fmt.Printf("Hello, %s!\n", *name)
}
```

Запуск:

```bash
go run main.go              # Hello, World!
go run main.go -name Alice  # Hello, Alice!
go run main.go --name Bob   # Hello, Bob! (двойной дефис тоже работает)
```

**Что здесь происходит:**

`flag.String` создаёт флаг с именем `name`, значением по умолчанию `World` и описанием. Она возвращает **указатель** на строку — поэтому обращаемся через `*name`.

### Все типы флагов

Для каждого базового типа есть своя функция:

```go
// Булевы флаги
debug := flag.Bool("debug", false, "Включить отладку")
verbose := flag.Bool("v", false, "Подробный вывод")

// Числа
port := flag.Int("port", 8080, "Порт сервера")
timeout := flag.Duration("timeout", 30*time.Second, "Таймаут")

// Строки
host := flag.String("host", "localhost", "Хост")
config := flag.String("config", "", "Путь к конфигу")

// Парсинг
flag.Parse()

// Использование
if *debug {
    log.Println("Debug mode on")
}
server := fmt.Sprintf("%s:%d", *host, *port)
```

**Важный момент с булевыми флагами:**

```bash
# Все эти записи работают:
go run main.go -debug      # true
go run main.go -debug=true # true
go run main.go -debug false # false (значение после =)
```

Но будь осторожен: `-debug false` без `=` может запутать парсер. Лучше всегда использовать `=` для ясности.

---

## 🔧 FlagVar: связь с существующими переменными

Иногда нужно связать флаг с уже существующей переменной, а не создавать новую. Для этого есть функции `*Var`:

```go
package main

import (
    "flag"
    "fmt"
)

var (
    host string
    port int
    debug bool
)

func init() {
    flag.StringVar(&host, "host", "localhost", "Хост сервера")
    flag.IntVar(&port, "port", 8080, "Порт")
    flag.BoolVar(&debug, "debug", false, "Отладка")
}

func main() {
    flag.Parse()
    
    // Используем напрямую, без указателей
    fmt.Printf("Server: %s:%d\n", host, port)
}
```

Разница в том, что `flag.String` возвращает указатель на новую переменную, а `flag.StringVar` связывает флаг с твоей переменной.

---

## 🎨 Пользовательские типы

Что если нужен флаг со сложным типом? Например, список значений или IP-адрес?

Пакет `flag` позволяет определить свой тип через интерфейс `Value`:

```go
type Value interface {
    String() string
    Set(string) error
}
```

### Пример: список строк

Хочешь передавать несколько значений: `-tag foo -tag bar -tag baz`?

```go
package main

import (
    "flag"
    "fmt"
    "strings"
)

// Тип для списка
type stringList []string

func (s *stringList) String() string {
    return strings.Join(*s, ", ")
}

func (s *stringList) Set(value string) error {
    *s = append(*s, value)
    return nil
}

func main() {
    var tags stringList
    
    flag.Var(&tags, "tag", "Теги (можно указывать несколько)")
    flag.Parse()
    
    fmt.Printf("Теги: %v\n", tags)
}
```

Запуск:

```bash
go run main.go -tag foo -tag bar -tag baz
# Теги: [foo bar baz]
```

**Как это работает:** `Set` вызывается каждый раз, когда встречается флаг. Поэтому `-tag foo -tag bar` добавит оба значения в слайс.

### Пример: IP-адрес

```go
package main

import (
    "flag"
    "fmt"
    "net"
)

func main() {
    var addr net.IP
    
    flag.TextVar(&addr, "addr", net.ParseIP("127.0.0.1"), "IP-адрес")
    flag.Parse()
    
    fmt.Printf("Адрес: %s\n", addr)
}
```

`TextVar` работает с типами, реализующими `encoding.TextUnmarshaler`. `net.IP` реализует этот интерфейс, поэтому парсинг работает автоматически.

---

## 🧩 FlagSet: изолированные наборы флагов

По умолчанию все флаги регистрируются в глобальном `flag.CommandLine`. Но что если нужно несколько независимых наборов? Например, для подкоманд?

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Используйте: serve или build")
        os.Exit(1)
    }
    
    switch os.Args[1] {
    case "serve":
        serveCmd := flag.NewFlagSet("serve", flag.ExitOnError)
        port := serveCmd.Int("port", 8080, "Порт")
        serveCmd.Parse(os.Args[2:])
        
        fmt.Printf("Запуск сервера на порту %d\n", *port)
        
    case "build":
        buildCmd := flag.NewFlagSet("build", flag.ExitOnError)
        output := buildCmd.String("o", "app", "Выходной файл")
        buildCmd.Parse(os.Args[2:])
        
        fmt.Printf("Сборка в %s\n", *output)
        
    default:
        fmt.Println("Неизвестная команда")
        os.Exit(1)
    }
}
```

Запуск:

```bash
go run main.go serve -port 3000   # Запуск сервера на порту 3000
go run main.go build -o myapp     # Сборка в myapp
```

**Зачем это нужно:** каждый `FlagSet` живёт своей жизнью. Флаги `serve` не конфликтуют с флагами `build`.

### Обработка ошибок в FlagSet

При создании `FlagSet` можно указать, как обрабатывать ошибки:

```go
// Продолжить даже при ошибке
fs := flag.NewFlagSet("test", flag.ContinueOnError)

// Вывести ошибку и выйти (по умолчанию)
fs := flag.NewFlagSet("test", flag.ExitOnError)

// Вызвать панику
fs := flag.NewFlagSet("test", flag.PanicOnError)
```

---

## 📋 Справка и помощь

Пакет `flag` автоматически генерирует справку:

```bash
go run main.go -h
go run main.go --help
```

Вывод:

```
Usage of main:
  -config string
        Путь к конфигу (default "config.yaml")
  -debug
        Включить отладку
  -host string
        Хост сервера (default "localhost")
  -port int
        Порт сервера (default 8080)
```

### Кастомная справка

Можно переопределить функцию вывода справки:

```go
flag.Usage = func() {
    fmt.Fprintf(os.Stderr, "Использование: %s [опции]\n\n", os.Args[0])
    fmt.Fprintf(os.Stderr, "Опции:\n")
    flag.PrintDefaults()
}
```

---

## ⚠️ Важные нюансы

### 1. Порядок имеет значение

`flag.Parse()` должна вызываться **после** объявления всех флагов и **до** использования значений:

```go
// ❌ Неправильно
flag.Parse()
name := flag.String("name", "", "Имя")  // слишком поздно!
fmt.Println(*name)  // паника или пустое значение

// ✅ Правильно
name := flag.String("name", "", "Имя")
flag.Parse()
fmt.Println(*name)
```

### 2. Флаги после `--`

Двойной дефис `--` останавливает парсинг флагов:

```bash
go run main.go -name Alice -- -not-a-flag
```

Всё после `--` считается позиционными аргументами, даже если начинается с `-`.

### 3. Позиционные аргументы

После `flag.Parse()` доступны позиционные аргументы:

```go
flag.Parse()

args := flag.Args()      // все позиционные аргументы
first := flag.Arg(0)     // первый аргумент
count := flag.NArg()     // количество аргументов
```

Пример:

```bash
go run main.go -verbose file1.txt file2.txt
```

```go
verbose := flag.Bool("verbose", false, "")
flag.Parse()

files := flag.Args()  // ["file1.txt", "file2.txt"]
```

### 4. Флаги не перезаписываются

Если указать флаг несколько раз, последнее значение **не** перезаписывает предыдущее — оно добавляется (для типов, которые это поддерживают):

```bash
go run main.go -tag foo -tag bar
```

Для `stringList` из примера выше это даст `[foo bar]`. Для простых типов (`int`, `string`) последнее значение побеждает.

### 5. Глобальное состояние

`flag.CommandLine` — глобальная переменная. Если ты пишешь библиотеку, не регистрируй флаги в `init()`:

```go
// ❌ Плохо в библиотеке
func init() {
    flag.Bool("debug", false, "")  // загрязняет глобальное пространство
}

// ✅ Лучше
func NewConfig() *Config {
    fs := flag.NewFlagSet("mylib", flag.ContinueOnError)
    debug := fs.Bool("debug", false, "")
    return &Config{debug: *debug}
}
```

---

## 🛠 Практические паттерны

### Паттерн 1: Конфигурация приложения

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "time"
)

type Config struct {
    Host    string
    Port    int
    Debug   bool
    Timeout time.Duration
}

func ParseConfig() *Config {
    cfg := &Config{}
    
    flag.StringVar(&cfg.Host, "host", "localhost", "Хост")
    flag.IntVar(&cfg.Port, "port", 8080, "Порт")
    flag.BoolVar(&cfg.Debug, "debug", false, "Отладка")
    flag.DurationVar(&cfg.Timeout, "timeout", 30*time.Second, "Таймаут")
    
    flag.Parse()
    return cfg
}

func main() {
    cfg := ParseConfig()
    fmt.Printf("Конфиг: %+v\n", cfg)
}
```

### Паттерн 2: Валидация флагов

```go
flag.Parse()

if *port < 1 || *port > 65535 {
    fmt.Fprintf(os.Stderr, "Порт должен быть от 1 до 65535\n")
    flag.Usage()
    os.Exit(1)
}

if *host == "" {
    fmt.Fprintf(os.Stderr, "Хост не может быть пустым\n")
    flag.Usage()
    os.Exit(1)
}
```

### Паттерн 3: Логирование в зависимости от флага

```go
flag.Parse()

if *debug {
    log.SetLevel(log.DebugLevel)
}

log.Debug("Отладочное сообщение")
log.Info("Информационное сообщение")
```

---

## 📊 Сравнение с альтернативами

| Библиотека | Плюсы | Минусы |
|------------|-------|--------|
| `flag` (stdlib) | Встроен, прост, нет зависимостей | Нет подкоманд, базовый help |
| `cobra` | Подкоманды, авто-completion, красивый help | Зависимость, сложнее |
| `kingpin` | Очень гибкий, валидация | Зависимость, больше кода |
| `urfave/cli` | Баланс между простотой и функциями | Зависимость |

**Когда использовать `flag`:**
- Простая утилита с несколькими флагами
- Не нужны подкоманды
- Хочешь минимум зависимостей

**Когда смотреть дальше:**
- Нужны подкоманды (`git commit`, `docker run`)
- Требуется авто-completion
- Сложная валидация и группировка флагов

---

## 🎯 Золотые правила

1. **Объявляй флаги до `Parse()`** — иначе они не зарегистрируются.
2. **Используй `*Var` для существующих переменных** — меньше указателей, чище код.
3. **Валидируй после парсинга** — `flag` не проверяет значения.
4. **Не регистрируй флаги в `init()` библиотек** — используй `FlagSet`.
5. **Пиши понятные описания** — они попадут в `-help`.
6. **Для сложных CLI смотри `cobra`** — `flag` хорош для простого.

---

Пакет `flag` — это как отвёртка. Не самая красивая, не самая умная. Но когда нужно закрутить винт — работает безотказно. Научись использовать его хорошо, и ты поймёшь, когда пора брать «шуруповёрт» вроде `cobra`.
