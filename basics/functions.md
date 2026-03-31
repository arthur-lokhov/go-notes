# 📦 Функции в Go

Представь, что ты строишь дом. Ты не пытаешься сделать всё сразу — разбиваешь работу на задачи: фундамент, стены, крыша, электрика. Каждая задача — это функция.

**Функция в Go** — это именованный блок кода, который решает одну задачу. Ты даёшь ей входные данные (параметры), она делает свою работу и возвращает результат.

Но функции в Go особенные. Они могут возвращать несколько значений, принимать переменное число аргументов, иметь именованные результаты. Давай разберёмся со всеми этими возможностями.

---

## 🔧 Базовый синтаксис

Функция в Go объявляется так:

```go
func name(parameter type) returnType {
    // тело функции
    return value
}
```

**Пример:**

```go
func add(a int, b int) int {
    return a + b
}
```

### Типы параметров

Каждый параметр должен иметь тип. Если несколько параметров одного типа, можно сократить:

```go
// Полная форма
func add(a int, b int) int {
    return a + b
}

// Сокращённая форма
func add(a, b int) int {
    return a + b
}
```

---

## 📤 Передача аргументов в функцию

### Передача по значению

В Go **все аргументы передаются по значению**. Это фундаментальное правило, которое нужно понимать.

**Что это значит:**

```go
func modify(x int) {
    x = 10  // изменяется локальная копия
}

a := 5
modify(a)
fmt.Println(a)  // 5, оригинал не изменился
```

**Что происходит в памяти:**

```
┌─────────────────────────────────────────────────────────────┐
│  До вызова:                                                 │
│  a = 5 (в main)                                             │
│                                                             │
│  Во время вызова:                                           │
│  a = 5 (в main)     x = 5 (копия в modify)                  │
│                     x = 10 (изменяется копия)               │
│                                                             │
│  После вызова:                                              │
│  a = 5 (в main)     x уничтожён                             │
└─────────────────────────────────────────────────────────────┘
```

**Для изменения используй указатель:**

```go
func modify(x *int) {
    *x = 10  // изменяется значение по адресу
}

a := 5
modify(&a)
fmt.Println(a)  // 10, оригинал изменился
```

**Когда использовать указатель:**

| Ситуация | Использовать указатель |
|----------|----------------------|
| Нужно изменить аргумент | ✅ Да |
| Большая структура (> 3-4 полей) | ✅ Да |
| Маленький тип (int, bool) | ❌ Нет |
| Не нужно изменять | ❌ Нет |

**Примеры:**

```go
// ✅ Маленький тип — по значению
func add(a, b int) int {
    return a + b
}

// ✅ Большая структура — по указателю
func process(user *User) error {
    user.LastLogin = time.Now()
    return save(user)
}

// ❌ Избыточно: int по указателю
func add(a, b *int) int {
    return *a + *b
}
```

### Слайсы и мапы: значение или ссылка?

**Технически:** слайсы и мапы передаются **по значению**.

**Практически:** они ведут себя как **ссылочные типы**.

**Почему:** заголовок слайса/мапы содержит указатель на данные.

```go
// Заголовок слайса (упрощённо)
type sliceHeader struct {
    Data uintptr  // указатель на данные
    Len  int      // длина
    Cap  int      // ёмкость
}
```

**Что это значит на практике:**

```go
func modifySlice(s []int) {
    s[0] = 99  // ← изменяет ОРИГИНАЛЬНЫЕ данные
}

slice := []int{1, 2, 3}
modifySlice(slice)
fmt.Println(slice)  // [99 2 3], изменилось!
```

**Визуализация:**

```
┌─────────────────────────────────────────────────────────────┐
│  slice (в main)         s (в modifySlice)                   │
│  ┌─────────────────┐     ┌─────────────────┐                │
│  │ Data ───────────┼────→│ Data (те же!)   │                │
│  │ Len  = 3        │     │ Len  = 3        │                │
│  │ Cap  = 3        │     │ Cap  = 3        │                │
│  └─────────────────┘     └─────────────────┘                │
│         ↓                          ↓                        │
│  ┌─────────────────────────────────────────┐                │
│  │  [1]  [2]  [3]  ← ОДНИ И ТЕ ЖЕ ДАННЫЕ   │                │
│  └─────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

**Но есть нюанс:**

```go
func appendOne(s []int) {
    s = append(s, 1)  // ← может создать НОВЫЙ слайс!
}

slice := []int{1, 2, 3}
appendOne(slice)
fmt.Println(slice)  // [1 2 3], не изменилось!
```

**Почему:** `append` может создать новый слайс с новым массивом данных.

**Когда слайс изменяется, а когда нет:**

```go
// ✅ Изменение элементов — видно
func modifyElements(s []int) {
    s[0] = 99
}

// ❌ Изменение длины/ёмкости — не видно
func appendElement(s []int) {
    s = append(s, 99)
}

// ✅ Возврат нового слайса — работает
func appendElement(s []int) []int {
    return append(s, 99)
}

slice := appendElement(slice)
```

**Мапы работают аналогично:**

```go
// ✅ Изменение мапы — видно
func modifyMap(m map[string]int) {
    m["key"] = 99
}

// ❌ Присваивание новой мапы — не видно
func replaceMap(m map[string]int) {
    m = make(map[string]int)  // новая мапа локально
}
```

---

## 🎁 Несколько возвращаемых значений

В Go функция может возвращать **несколько значений**. Это не кортеж, не структура — это нативная возможность языка.

```go
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль")
    }
    return a / b, nil
}

// Использование
result, err := divide(10, 2)
```

### Зачем это нужно?

**1. Обработка ошибок:**

```go
// ❌ В других языках: исключения или коды ошибок
try {
    result = divide(10, 2)
} catch (Exception e) {
    // обработка
}

// ✅ В Go: ошибка как обычное значение
result, err := divide(10, 2)
if err != nil {
    // обработка
}
```

**2. Возврат значения и флага:**

```go
func findUser(id int) (*User, bool) {
    user, ok := users[id]
    return user, ok
}

user, found := findUser(123)
if !found {
    fmt.Println("Не найден")
}
```

### Сколько максимум можно возвращать?

**Технического ограничения нет** — синтаксис Go позволяет вернуть сколько угодно значений:

```go
func manyReturns() (int, int, int, int, int, int, int, int, int, int) {
    return 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
}
```

**Но есть практические ограничения:**

| Аспект | Ограничение |
|--------|-------------|
| **Спецификация Go** | Нет явного лимита |
| **Компилятор** | Зависит от доступной памяти стека |
| **Стиль кода** | 2-3 значения (идиоматично) |
| **Читаемость** | Падает после 4-5 значений |

**Когда много значений — это плохо:**

```go
// ❌ Плохо: непонятно что есть что
func getData() (int, string, bool, int, string, error) {
    // ...
}

id, name, ok, count, email, err := getData()
// Что есть что? Придётся смотреть в документацию
```

**Лучшие альтернативы:**

**1. Структура:**

```go
// ✅ Лучше
type Result struct {
    ID    int
    Name  string
    OK    bool
    Count int
    Email string
    Err   error
}

func getData() Result {
    // ...
}

result := getData()
fmt.Println(result.ID, result.Name)
```

**2. Группировка по смыслу:**

```go
// ✅ Лучше
type UserInfo struct {
    Name  string
    Email string
    OK    bool
}

type Stats struct {
    ID    int
    Count int
}

func getData() (UserInfo, Stats, error) {
    // ...
}

info, stats, err := getData()
```

**Идиоматичное количество:**

| Количество | Когда уместно |
|------------|---------------|
| **1** | Обычный случай |
| **2** | `value, err` или `value, ok` |
| **3** | `value, err, extra` (редко) |
| **4+** | Почти всегда лучше структуру |

**Практическое правило:**

> **Если возвращаешь больше 3 значений — используй структуру.**

**Почему:**

- Именованные поля понятнее
- Легче добавлять новые поля
- Не нужно менять сигнатуры везде
- Проще передавать в другие функции

### Игнорирование значений

Если не нужно какое-то значение, используй `_`:

```go
result, _ := divide(10, 2)  // игнорируем ошибку
_, err := divide(10, 0)     // игнорируем результат
```

---

## 🏷️ Именованные возвращаемые значения

Функция может дать имена возвращаемым значениям. Они становятся переменными внутри функции.

```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return  // голый return, возвращает x и y
}
```

### Как это работает

Именованные результаты — это переменные, которые:

1. Создаются при входе в функцию
2. Инициализируются нулевыми значениями
3. Возвращаются при `return` без значений

```go
func example() (result int, err error) {
    // result = 0, err = nil изначально
    
    result = 42
    // err всё ещё nil
    
    return  // возвращает (42, nil)
}
```

### Когда использовать

**✅ Хорошо для коротких функций:**

```go
func parseConfig(s string) (key, value string, err error) {
    parts := strings.Split(s, "=")
    if len(parts) != 2 {
        err = fmt.Errorf("неверный формат")
        return
    }
    key, value = parts[0], parts[1]
    return
}
```

**❌ Плохо для длинных функций:**

```go
// ❌ Трудно отследить, где и что меняется
func complexFunction() (result int, err error) {
    // 50 строк кода...
    result = 1
    // ещё 50 строк...
    err = someError
    // ещё код...
    return
}
```

### Голый return (naked return)

`return` без значений возвращает именованные результаты:

```go
func example() (x int) {
    x = 5
    return  // возвращает 5
}
```

**Предостережение:** голый `return` может запутать. Лучше явно указывать значения:

```go
// ✅ Яснее
func example() (x int) {
    x = 5
    return x
}
```

---

## 📮 Variadic функции (переменное число аргументов)

**Variadic функция** принимает переменное число аргументов.

```go
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}

// Вызов
sum(1, 2, 3)        // 6
sum(1, 2, 3, 4, 5)  // 15
sum()               // 0
```

### Как это работает внутри

`...int` превращается в слайс `[]int`:

```go
func sum(numbers ...int) int {
    // numbers имеет тип []int
    fmt.Printf("%T\n", numbers)  // []int
}
```

### Передача слайса как variadic

Если у тебя уже есть слайс, используй `...` для распаковки:

```go
numbers := []int{1, 2, 3, 4, 5}
sum(numbers...)  // распаковывает слайс
```

### Variadic с другими параметрами

Variadic параметр должен быть последним:

```go
// ✅ Правильно
func printf(format string, args ...interface{}) {
    // ...
}

// ❌ Неправильно
func wrong(args ...interface{}, format string) {
    // ошибка компиляции
}
```

### Пустой variadic

Если не передать аргументы, будет пустой слайс (не `nil`):

```go
func show(args ...int) {
    fmt.Printf("len=%d, nil=%v\n", len(args), args == nil)
}

show()  // len=0, nil=false
```

---

## 🔁 Рекурсия

Функция может вызывать сама себя:

```go
func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}
```

### Ограничения

**Стек вызовов ограничен.** Бесконечная рекурсия приведёт к переполнению стека:

```go
func infinite() {
    infinite()  // stack overflow
}
```

### Хвостовая рекурсия

Go **не оптимизирует** хвостовую рекурсию. Каждый вызов добавляет кадр (свой отдельный контекст выполнения) в стек.

```go
// Хвостовая рекурсия, но не оптимизируется
func sum(n, acc int) int {
    if n == 0 {
        return acc
    }
    return sum(n-1, acc+n)  // хвостовой вызов
}
```

Для больших чисел лучше использовать итерацию:

```go
func sum(n int) int {
    total := 0
    for i := 1; i <= n; i++ {
        total += i
    }
    return total
}
```

---

## 🔐 Замыкания (Closures)

**Замыкание** — это функция, которая «помнит» переменные из внешней области, даже если она вызывается вне этой области.

```go
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y  // x «захвачена» из внешней области
    }
}

add5 := makeAdder(5)
fmt.Println(add5(3))  // 8
fmt.Println(add5(10)) // 15
```

### Практическое применение замыканий

**1. Фабрика функций:**

```go
func makeMultiplier(factor int) func(int) int {
    return func(x int) int {
        return x * factor
    }
}

double := makeMultiplier(2)
triple := makeMultiplier(3)

fmt.Println(double(5))  // 10
fmt.Println(triple(5))  // 15
```

**2. Частичное применение (каррирование):**

```go
func add(a, b int) int {
    return a + b
}

func makeAdd5() func(int) int {
    return func(b int) int {
        return add(5, b)
    }
}

add5 := makeAdd5()
fmt.Println(add5(3))  // 8
```

**3. Инкапсуляция состояния:**

```go
func makeBankAccount() (func() int, func(int)) {
    balance := 0
    
    getBalance := func() int {
        return balance
    }
    
    deposit := func(amount int) {
        balance += amount
    }
    
    return getBalance, deposit
}

get, deposit := makeBankAccount()
deposit(100)
fmt.Println(get())  // 100
```

**4. Логирование/профилирование:**

```go
func withLogging(fn func(int) int) func(int) int {
    return func(x int) int {
        fmt.Printf("Вызов функции с %d\n", x)
        result := fn(x)
        fmt.Printf("Результат: %d\n", result)
        return result
    }
}

fn := withLogging(func(x int) int {
    return x * 2
})

fn(5)
// Вызов функции с 5
// Результат: 10
```

---

## 🛡️ Defer, Panic, Recover

**`defer`**, **`panic`**, **`recover`** — три механизма для управления выполнением функции.

### defer: отложенное выполнение

**`defer`** откладывает выполнение функции до момента завершения окружающей функции.

```go
func processFile(filename string) {
    file, err := os.Open(filename)
    if err != nil {
        return
    }
    defer file.Close()  // закроется, когда processFile завершится

    // обработка файла
}
```

**Как работает defer:**

```
┌─────────────────────────────────────────────────────────────┐
│  Когда Go встречает defer:                                  │
│                                                             │
│  1. Сразу вычисляет аргументы (но не вызывает функцию)      │
│  2. Запоминает вызов в стек отложенных вызовов              │
│  3. Выполняет отложенное, когда функция завершается         │
└─────────────────────────────────────────────────────────────┘
```

**Порядок выполнения — LIFO (Last In, First Out):**

```go
func example() {
    defer fmt.Println("Первый")
    defer fmt.Println("Второй")
    fmt.Println("Начало")
}

// Вывод:
// Начало
// Второй
// Первый
```

**Аргументы вычисляются сразу:**

```go
func example() {
    x := 5
    defer fmt.Println("x =", x)  // запомнит x = 5
    x = 10
    // Выведет: x = 5, а не x = 10
}
```

Если нужно вычислить аргументы при выполнении defer, используй замыкание:

```go
func example() {
    x := 5
    defer func() {
        fmt.Println("x =", x)  // вычислит x при выполнении
    }()
    x = 10
    // Выведет: x = 10
}
```

**Практическое применение:**

```go
// 1. Закрытие ресурсов
func processFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()  // гарантированно закроется
}

// 2. Разблокировка мьютекса
mu.Lock()
defer mu.Unlock()

// 3. Логирование времени выполнения
func slowFunction() {
    start := time.Now()
    defer func() {
        fmt.Printf("Выполнялось %v\n", time.Since(start))
    }()
}
```

**defer в циклах — осторожно:**

```go
// ❌ Плохо: все файлы закроются только в конце функции
for _, filename := range files {
    file, err := os.Open(filename)
    defer file.Close()  // все Close отложены
}

// ✅ Лучше: анонимная функция
for _, filename := range files {
    func() {
        file, err := os.Open(filename)
        if err != nil {
            return
        }
        defer file.Close()  // закроется в конце анонимной функции
    }()
}
```

---

### panic: аварийное завершение

**`panic`** — это способ сказать: «что-то пошло не так, я не могу продолжить».

```go
if err != nil {
    panic(err)
}
```

**Что происходит при панике:**

```
┌─────────────────────────────────────────────────────────────┐
│  main()                                                     │
│    ↓                                                        │
│  processFile()                                              │
│    ↓                                                        │
│  readFile()                                                 │
│    ↓                                                        │
│  panic! ← здесь началась паника                             │
│    ↑                                                        │
│  defer в readFile()  ← выполняются все defer                │
│    ↑                                                        │
│  defer в processFile()                                      │
│    ↑                                                        │
│  defer в main()                                             │
│    ↑                                                        │
│  Выход из программы                                         │
└─────────────────────────────────────────────────────────────┘
```

**Когда использовать panic:**

| ✅ Допустимо | ❌ Неприемлемо |
|--------------|----------------|
| Невосстановимая ошибка | Ожидаемые ошибки |
| Программистская ошибка | Валидация пользовательского ввода |
| В тестах (`t.Fatal`) | В продакшене для обычных ошибок |

**Примеры:**

```go
// ✅ Невосстановимая ошибка
func init() {
    config, err := loadConfig()
    if err != nil {
        panic(fmt.Sprintf("Не удалось загрузить конфиг: %v", err))
    }
}

// ✅ Программистская ошибка
func NewServer(port int) *Server {
    if port < 1 || port > 65535 {
        panic(fmt.Sprintf("Неверный порт: %d", port))
    }
}

// ❌ Ожидаемая ошибка
file, err := os.Open("config.json")
if err != nil {
    panic(err)  // файл может отсутствовать — это нормально
}
```

**panic vs error:**

| panic | error |
|-------|-------|
| Невосстановимая ошибка | Ожидаемая ошибка |
| Завершает выполнение | Возвращается как значение |
| Для программистских ошибок | Для ошибок времени выполнения |
| Использовать редко | Использовать всегда |

---

### recover: восстановление после паники

**`recover`** позволяет поймать панику и предотвратить завершение программы.

```go
func safeFunction() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Поймана паника:", r)
        }
    }()

    panic("что-то пошло не так")
}
```

**Важно:** `recover` работает **только внутри defer**:

```go
// ❌ Бесполезно
recover()  // всегда вернёт nil
panic("test")

// ✅ Правильно
defer func() {
    r := recover()  // поймает панику
    if r != nil {
        fmt.Println("Поймано:", r)
    }
}()
panic("test")
```

**Практическое применение:**

```go
// 1. Защита сервера от падения
func handleRequest(w http.ResponseWriter, r *http.Request) {
    defer func() {
        if err := recover(); err != nil {
            log.Printf("Паника в обработчике: %v", err)
            http.Error(w, "Внутренняя ошибка", http.StatusInternalServerError)
        }
    }()

    // обработка запроса
}

// 2. Тестирование паник
func TestPanic(t *testing.T) {
    defer func() {
        if r := recover(); r == nil {
            t.Error("Ожидалась паника")
        }
    }()

    functionThatPanics()
}
```

**Важные нюансы:**

```go
// 1. defer выполняется даже при панике
func example() {
    defer fmt.Println("Это выполнится")
    panic("паника")
}

// 2. recover возвращает значение паники
defer func() {
    r := recover()
    fmt.Printf("Тип: %T, Значение: %v\n", r, r)
}()
panic("строка")      // r = "строка"
panic(fmt.Errorf("ошибка"))  // r = error
panic(42)            // r = 42

// 3. recover только в той же горутине
go func() {
    panic("test")
}()
// recover() здесь не поймает панику
```

---

## 🧬 Внутреннее устройство: как компилятор видит функции

Когда компилятор обрабатывает функцию, он создаёт несколько представлений.

### Представление функции в компиляторе

В компиляторе Go функция представлена структурой `types.Func`:

```go
type Func struct {
    object       // базовая структура (имя, тип, позиция)
    hasPtrRecv_ bool  // есть ли указательный receiver (для методов)
    origin      *Func // если не nil, функция, из которой эта была инстанцирована
}
```

**Поля:**

| Поле | Тип | Описание |
|------|-----|----------|
| `object` | `object` | Базовая структура с именем, типом, позицией |
| `hasPtrRecv_` | `bool` | true, если у метода receiver — указатель (например, `*T`) |
| `origin` | `*Func` | Для generics: оригинальная функция, из которой создана эта |

**Пример:**

```go
func add(a int, b int) (result int, err error) {
    return a + b, nil
}
```

**В компиляторе:**

```
Func "add":
    object:
        name: "add"
        type: func(int, int) (int, error)
        pos: main.go:5:6
    hasPtrRecv_: false  // не метод
    origin: nil  // не generic
```

**Для методов:**

```go
type Counter struct{}

func (c *Counter) Inc() {  // ← receiver указатель
    // ...
}
```

**В компиляторе:**

```
Func "Inc":
    object:
        name: "Inc"
        type: func(*Counter)
        pos: main.go:10:1
    hasPtrRecv_: true  // receiver — указатель
    origin: nil
```

**Для generics (Go 1.18+):**

```go
func Map[T, U any](s []T, f func(T) U) []U {
    // ...
}

// При вызове создаётся инстанциация:
Map[int, string](nums, strconv.Itoa)
```

**В компиляторе:**

```
Func "Map":
    object:
        name: "Map"
        type: func([]T, func(T) U) []U
    hasPtrRecv_: false
    origin: nil  // оригинальная generic функция

Func "Map[int, string]":
    object:
        name: "Map"
        type: func([]int, func(int) string) []string
    hasPtrRecv_: false
    origin: → Func "Map"  // инстанциация из оригинальной
```

### Closure (замыкание)

Замыкание в Go компилятор может представить **двумя способами** в зависимости от ситуации.

**1. Оптимизация: прямой вызов (без аллокации)**

Если замыкание вызывается сразу после создания, компилятор **не создаёт объект замыкания**. Вместо этого захваченные переменные передаются как аргументы:

```go
// Исходный код
x := 10
func() {
    println(x)  // x захвачена
}()

// После оптимизации (directClosureCall)
// Компилятор добавляет параметр-указатель для захваченных переменных
func(x *int) {
    println(*x)  // x передаётся как аргумент-указатель
}(&x)
```

**Как это работает (из исходников компилятора):**

```go
// cmd/compile/internal/walk/closure.go
func directClosureCall(n *ir.CallExpr) {
    // Для каждого захваченного переменного:
    for _, v := range clofn.ClosureVars {
        if !v.Byval() {
            // Если захвачено по ссылке → добавляется параметр-указатель
            // v.Type() — это T, а параметр становится *T
            addr := ir.NewNameAt(..., types.NewPtr(v.Type()))
            v.Heapaddr = addr
        }
        // Добавляется как параметр функции
        params = append(params, fld)
    }
}
```

**2. Обычный случай: структура с функцией и переменными**

Если замыкание сохраняется или передаётся, создаётся **структура**:

```go
// Исходный код
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}

// В компиляторе (walkClosure)
// Создаётся структура:
type closure struct {
    Func uintptr  // указатель на функцию
    X    int      // захваченная переменная
}

// Возвращается указатель на структуру:
return &closure{Func: makeAdder·f, X: 5}
```

**Как это работает (из исходников компилятора):**

```go
// cmd/compile/internal/walk/closure.go
func walkClosure(clo *ir.ClosureExpr, init *ir.Nodes) ir.Node {
    // Создаётся тип замыкания
    typ := typecheck.ClosureType(clo)
    
    // Создаётся композитный литерал (структура)
    clos := ir.NewCompLitExpr(..., ir.OCOMPLIT, typ, nil)
    
    // Заполняется: функция + захваченные переменные
    clos.List = append([]ir.Node{
        ir.NewUnaryExpr(..., ir.OCFUNC, clofn.Nname),  // функция
    }, closureArgs(clo)...)  // захваченные переменные
    
    // Возвращается указатель на структуру
    addr := typecheck.NodAddr(clos)
    return walkExpr(addr, init)
}
```

**Захват по значению vs по ссылке:**

Замыкания захватывают переменные:
- по значению (если не изменяются)
- по ссылке (если изменяются или адрес нужен)

```go
func makeClosure() func() {
    x := 10  // захватывается по значению
    y := 20  // захватывается по ссылке (изменяется)
    
    return func() {
        println(x)  // читается копия
        y++         // изменяется оригинал
    }
}
```

**В компиляторе:**

```go
// closureArgs возвращает выражения для инициализации
func closureArgs(clo *ir.ClosureExpr) []ir.Node {
    args := make([]ir.Node, len(fn.ClosureVars))
    for i, v := range fn.ClosureVars {
        outer := v.Outer
        if !v.Byval() {
            // Если не по значению → берётся адрес
            outer = typecheck.NodAddrAt(fn.Pos(), outer)
        }
        args[i] = typecheck.Expr(outer)
    }
    return args
}
```

**Визуализация:**

```
┌─────────────────────────────────────────────────────────────┐
│  makeAdder(5)                                               │
│     ↓                                                       │
│  Компилятор проверяет: вызывается сразу?                    │
│     ├─ Да → directClosureCall                               │
│     │   └─ x передаётся как аргумент (без аллокации)        │
│     │                                                       │
│     └─ Нет → walkClosure                                    │
│         └─ Создаётся структура:                             │
│             ┌─────────────────────────────────┐             │
│             │  Func: makeAdder·f              │             │
│             │  X: 5                           │             │
│             └─────────────────────────────────┘             │
│         └─ Возвращается &closure                            │
└─────────────────────────────────────────────────────────────┘
```

**Вложенные замыкания:**

Для вложенных замыканий каждое создаёт **свою структуру**:

```go
func makeAdder(x int) func(int) func(int) int {
    return func(y int) func(int) int {  // ← Замыкание A
        return func(z int) int {  // ← Замыкание B
            return x + y + z
        }
    }
}

// В компиляторе создаются ДВЕ структуры:

// Для замыкания A (захватывает x)
type closureA struct {
    Func uintptr  // указатель на функцию A
    X    int     // захваченная переменная x
}

// Для замыкания B (захватывает x и y)
type closureB struct {
    Func uintptr  // указатель на функцию B
    X    int     // захваченная переменная x
    Y    int     // захваченная переменная y
}

// Использование:
adder := makeAdder(1)      // создаётся closureA{X: 1}
add5 := adder(5)           // создаётся closureB{X: 1, Y: 5}
result := add5(10)         // возвращает 1 + 5 + 10 = 16
```

**Визуализация вложенных замыканий:**

```
┌─────────────────────────────────────────────────────────────┐
│  makeAdder(1)                                               │
│     ↓                                                       │
│  closureA: {Func: A·f, X: 1}                                │
│     ↓ возвращает func(int) func(int) int                    │
│  adder(5)                                                   │
│     ↓                                                       │
│  closureB: {Func: B·f, X: 1, Y: 5}  ← копирует X из closureA│
│     ↓ возвращает func(int) int                              │
│  add5(10)                                                   │
│     ↓                                                       │
│  1 + 5 + 10 = 16                                            │
└─────────────────────────────────────────────────────────────┘
```

### Variadic функции внутри

```go
func sum(nums ...int) int {
    // ...
}
```

**В компиляторе:**

```go
// Внутри функции
nums имеет тип []int  // слайс

// При вызове
sum(1, 2, 3)
// Компилятор создаёт слайс: []int{1, 2, 3}
```

**Представление:**

```
Func "sum":
    prms: (nums []int)  // variadic превращается в слайс
    signature: func(...int) int
```

### Именованные результаты

```go
func example() (result int, err error) {
    result = 42
    return  // naked return
}
```

**В компиляторе:**

```
Func "example":
    results: (result int, err error)
    
    Тело функции:
        result = 0  // инициализация нулевыми
        err = nil
        result = 42
        return result, err  // naked return разворачивается
```

**Naked return разворачивается в явный:**

```go
// Исходный код
return

// После разворачивания
return result, err
```

### Stack Frame: кадр стека

Когда функция вызывается, создаётся **кадр стека**:

```
┌─────────────────────────────────────────────────────────────┐
│  Stack Frame для функции                                    │
├─────────────────────────────────────────────────────────────┤
│  Аргументы (входные)                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  a (int) = 5                                        │    │
│  │  b (int) = 10                                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Локальные переменные                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  temp (int) = 0                                     │    │
│  │  result (int) = 0                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Возвращаемые значения (выходные)                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  return0 (int) = ?                                  │    │
│  │  return1 (error) = ?                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  Служебные данные                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  return address                                     │    │
│  │  saved frame pointer                                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Внутреннее устройство: defer, panic, recover

### defer: как работает внутри

**Два способа реализации defer:**

1. **Классический (defer chain)** — создаётся запись `_defer` в куче
2. **Open-coded** — для функций без defer в циклах, информация хранится в стеке

**Структура `_defer` (из runtime/runtime2.go):**

```go
type _defer struct {
    sp          uintptr  // стек-пойнтер вызвавшей функции
    pc          uintptr  // PC (Program Counter) вызвавшей функции
    fn          func()   // отложенная функция
    heap        bool     // выделен в куче?
    rangefunc   bool     // для range-over-func
    link        *_defer  // следующая запись в списке
}
```

**Как работает (из runtime/panic.go):**

```go
// deferproc — создание defer записи
func deferproc(fn func()) {
    gp := getg()
    d := newdefer()  // выделяется из per-P пула
    d.link = gp._defer
    gp._defer = d    // добавляется в голову списка
    d.fn = fn
    d.sp = sys.GetCallerSP()
    d.pc = sys.GetCallerPC()
}

// deferreturn — выполнение defer при выходе из функции
func deferreturn() {
    gp := getg()
    for {
        d := gp._defer
        if d == nil {
            break
        }
        gp._defer = d.link  // удаляется из списка
        d.fn()              // вызывается функция
        freedefer(d)        // возвращается в пул
    }
}
```

**Per-P пул для defers:**

```go
// newdefer — выделение defer записи
func newdefer() *_defer {
    pp := getg().m.p.ptr()
    if len(pp.deferpool) > 0 {
        // Берётся из пула (быстро, без аллокации)
        d := pp.deferpool[len(pp.deferpool)-1]
        pp.deferpool = pp.deferpool[:len(pp.deferpool)-1]
        return d
    }
    // Если пул пуст — аллокация
    return new(_defer)
}
```

**Визуализация списка defer:**

```
┌─────────────────────────────────────────────────────────────┐
│  gp._defer → d3 → d2 → d1 → nil                             │
│                 ↑                                           │
│  При deferreturn: d3 вызывается, gp._defer = d2             │
└─────────────────────────────────────────────────────────────┘
```

**Open-coded defers (оптимизация):**

Для функций без defer в циклах компилятор генерирует код без аллокации:

```go
// Исходный код
func f() {
    defer f1()
    defer f2()
    defer f3()
}

// В компиляторе (cmd/compile/internal/walk/stmt.go):
case ir.ODEFER:
    // Если нет defer в циклах и < 8 defer:
    // → open-coded defers
    // Информация хранится в стеке:
    // - bitmask: какие defer активны
    // - slots: указатели на функции
```

**Когда open-coded disallowed:**

```go
// cmd/compile/internal/walk/stmt.go
if ir.CurFunc.NumDefers > 8 {
    // Слишком много defer → классический способ
    ir.CurFunc.SetOpenCodedDeferDisallowed(true)
}
if n.Esc() != ir.EscNever {
    // defer в цикле → классический способ
    ir.CurFunc.SetOpenCodedDeferDisallowed(true)
}
```

---

### panic: как работает внутри

**Структура `_panic` (из runtime/runtime2.go):**

```go
type _panic struct {
    arg     any     // аргумент паники
    gopanicFP uintptr  // стек-пойнтер gopanic
    link    *_panic // следующая паника в стеке
    g       *g      // горутина, в которой произошла паника
}
```

**Как работает gopanic (из runtime/panic.go):**

```go
func gopanic(e any) {
    gp := getg()
    
    // Создаётся объект паники
    var p _panic
    p.arg = e
    p.link = gp._panic
    gp._panic = &p
    
    // Итерация по defer
    for {
        fn, ok := p.nextDefer()
        if !ok {
            break  // defer закончились
        }
        fn()  // вызывается defer
    }
    
    // Если recover не был вызван → фатальная ошибка
    fatalpanic(&p)
}
```

**Поиск следующего defer (nextDefer):**

```go
func (p *_panic) nextDefer() (func(), bool) {
    gp := getg()
    
    // Проверяем open-coded defers
    for p.deferBitsPtr != nil {
        bits := *p.deferBitsPtr
        if bits == 0 {
            p.deferBitsPtr = nil
            break
        }
        // Находим активный бит
        i := 7 - uintptr(sys.LeadingZeros8(bits))
        *p.deferBitsPtr &^= 1 << i  // очищаем бит
        return *(*func())(add(p.slotsPtr, i*goarch.PtrSize)), true
    }
    
    // Проверяем linked defers
    if d := gp._defer; d != nil && d.sp == uintptr(p.sp) {
        fn := d.fn
        popDefer(gp)  // удаляет defer
        return fn, true
    }
    
    // Переходим к следующему фрейму
    if !p.nextFrame() {
        return nil, false
    }
}
```

**Визуализация panic flow:**

```
┌─────────────────────────────────────────────────────────────┐
│  main()                                                     │
│    defer m1()                                               │
│    ↓                                                        │
│  processFile()                                              │
│    defer p1()                                               │
│    defer p2()                                               │
│    ↓                                                        │
│  readFile()                                                 │
│    defer r1()                                               │
│    ↓                                                        │
│  panic!("error") ← p.arg = "error"                          │
│    ↑                                                        │
│  gp._panic = &p                                             │
│    ↑                                                        │
│  p.nextDefer() → r1() → p2() → p1() → m1()                  │
│    ↑                                                        │
│  Если recover не вызван → fatalpanic()                      │
└─────────────────────────────────────────────────────────────┘
```

**panicCheck: защита runtime:**

```go
// panicCheck1 — превращает panic в throw если в runtime
func panicCheck1(pc uintptr, msg string) {
    if strings.HasPrefix(funcname(findfunc(pc)), "runtime.") {
        throw(msg)  // не panic, а throw!
    }
    gp := getg()
    if gp.m.mallocing != 0 {
        throw(msg)  // во время malloc → throw
    }
}
```

**Типы паник:**

```go
// Разные entry points для разных типов паник
func goPanicIndex(x int, y int)      // index out of range
func panicdivide()                    // divide by zero
func panicshift()                     // negative shift
func panicoverflow()                  // integer overflow
func panicmem()                       // nil pointer dereference
```

---

### recover: как работает внутри

**Алгоритм recover (из runtime/panic.go):**

```go
func gorecover() any {
    gp := getg()
    p := gp._panic
    
    // Нет паники или уже recovered
    if p == nil || p.goexit || p.recovered {
        return nil
    }
    
    // Проверяем, что recover вызван из defer
    canRecover := false
    systemstack(func() {
        var u unwinder
        u.init(gp, 0)
        u.next()  // skip systemstack_switch
        u.next()  // skip gorecover
        u.next()  // skip wrapper (если есть)
        
        // Должен быть ровно один non-wrapper фрейм
        // между gopanic и gorecover
        nonWrapperFrames := 0
        for ; u.valid(); u.next() {
            for iu, f := newInlineUnwinder(u.frame.fn, u.symPC()); f.valid(); f = iu.next(f) {
                sf := iu.srcFunc(f)
                switch sf.funcID {
                case abi.FuncIDWrapper:
                    continue  // wrapper не считаем
                case abi.FuncID_gopanic:
                    if u.frame.fp == uintptr(p.gopanicFP) && nonWrapperFrames > 0 {
                        canRecover = true
                    }
                    break loop
                default:
                    nonWrapperFrames++
                    if nonWrapperFrames > 1 {
                        break loop  // слишком много фреймов
                    }
                }
            }
        }
    })
    
    if !canRecover {
        return nil
    }
    
    // Успешный recover
    p.recovered = true
    return p.arg
}
```

**Проверка «правильного» recover:**

```
┌─────────────────────────────────────────────────────────────┐
│  ✅ Правильно:                                              │
│  foo()                                                      │
│    defer bar()                                              │
│    panic("panic")                                           │
│                                                             │
│  Стек: foo → gopanic → bar → gorecover                      │
│  nonWrapperFrames = 1 (bar) → canRecover = true             │
│                                                             │
│  ❌ Неправильно (слишком глубоко):                          │
│  foo()                                                      │
│    defer func() { func() { recover() }() }()                │
│    panic("panic")                                           │
│                                                             │
│  Стек: foo → gopanic → wrapper → func → func → gorecover    │
│  nonWrapperFrames = 2 → canRecover = false                  │
│                                                             │
│  ❌ Неправильно (defer recover()):                          │
│  foo()                                                      │
│    defer recover()                                          │
│    panic("panic")                                           │
│                                                             │
│  Стек: foo → gopanic → wrapper → gorecover                  │
│  nonWrapperFrames = 0 → canRecover = false                  │
└─────────────────────────────────────────────────────────────┘
```

**recovery: продолжение после recover:**

```go
// recovery — продолжает выполнение после recover
func recovery(gp *g) {
    p := gp._panic
    
    // Находим frame, где был вызван defer
    f := findfunc(p.retpc)
    gotoPc := f.entry() + uintptr(f.deferreturn)
    
    // Восстанавливаем стек
    gp.sched.sp = sp
    gp.sched.pc = gotoPc  // переходим на deferreturn
    gogo(&gp.sched)       // продолжаем выполнение
}
```

---

## 🎯 Итог

Функции в Go мощнее, чем кажутся на первый взгляд.

**Запомни:**

- Несколько возвращаемых значений — стандарт для обработки ошибок
- Именованные результаты — для коротких функций, с осторожностью
- Variadic параметры — для функций вроде `fmt.Println`
- Все аргументы передаются по значению
- Замыкания захватывают переменные по ссылке
- `defer` — для очистки ресурсов (LIFO)
- `panic` — только для невосстановимых ошибок
- `recover` — только внутри `defer`

**Избегай:**

- Длинных функций с именованными результатами
- Голого `return` в сложных функциях
- Бесконечной рекурсии
- `panic` для ожидаемых ошибок
- `recover` без веской причины
- `defer` в циклах без анонимной функции

Понимание функций — основа написания идиоматичного кода на Go.
