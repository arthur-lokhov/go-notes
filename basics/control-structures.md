# 🎛 Управляющие конструкции в Go

Представь, что ты дирижёр оркестра. У тебя есть партитура (код), и ты решаешь, какие инструменты играть, когда повторять пасаж, а когда перейти к следующей части.

**Управляющие конструкции** — это твои дирижёрские жесты. Они говорят программе: «если произошло это — сделай то», «повторяй, пока не закончится», «выбери один вариант из нескольких».

В Go управляющие конструкции проще, чем в многих языках. Нет `while`, нет `do-while`, нет тернарного оператора. Но того, что есть, достаточно для любых задач.

---

## 🚦 if: условное выполнение

**`if`** — это способ сказать: «если условие истинно, выполни этот код».

```go
if x > 0 {
    fmt.Println("Положительное")
}
```

### Базовый синтаксис

В Go `if` выглядит так:

```go
if условие {
    // код, если условие истинно
}
```

**Важно:** условие не обязательно заключать в скобки. Это не `if (x > 0)`, а просто `if x > 0`.

### else: альтернатива

```go
if x > 0 {
    fmt.Println("Положительное")
} else {
    fmt.Println("Не положительное")
}
```

### else if: цепочка условий

```go
if x > 0 {
    fmt.Println("Положительное")
} else if x < 0 {
    fmt.Println("Отрицательное")
} else {
    fmt.Println("Ноль")
}
```

### Инициализация в условии

Go позволяет объявить переменную прямо в `if`. Она видна только внутри `if`, `else if` и `else`.

```go
if x := getValue(); x > 0 {
    fmt.Println("Положительное:", x)
}
// x здесь недоступна
```

**Зачем это нужно:** переменная нужна только для проверки. Не засоряем внешнюю область видимости.

**Пример из практики:**

```go
if err := file.Close(); err != nil {
    log.Printf("Ошибка закрытия: %v", err)
}
```

### Подводный камень: вложенные if

```go
// ❌ Плохо: пирамида ужаса
if user != nil {
    if user.Active {
        if user.Balance > 0 {
            // код
        }
    }
}

// ✅ Лучше: ранний возврат
if user == nil {
    return
}
if !user.Active {
    return
}
if user.Balance <= 0 {
    return
}
// код
```

---

## 🔄 for: циклы

В Go только один вид цикла — **`for`**. Но он универсален.

### Базовый цикл

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

Это как `for` в C или Java: инициализация, условие, шаг.

### for как while

В Go нет `while`, но `for` может его заменить:

```go
// Как while
for condition {
    // код
}

// Бесконечный цикл
for {
    // код
}

// Бесконечный цикл с выходом
for {
    if done {
        break
    }
}
```

**Пример:**

```go
// Читаем, пока не конец файла
for buffer := readChunk(); buffer != nil; buffer = readChunk() {
    process(buffer)
}

// Или проще
for {
    buffer := readChunk()
    if buffer == nil {
        break  // выход из цикла
    }
    process(buffer)
}
```

### break и continue

**`break`** — выход из цикла:

```go
for i := 0; i < 10; i++ {
    if i == 7 {
        break  // выйти из цикла на 7
    }
    fmt.Println(i)  // 0, 1, 2, 3, 4, 5, 6
}
```

**`continue`** — пропустить текущую итерацию:

```go
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue  // пропустить чётные
    }
    fmt.Println(i)  // 1, 3, 5, 7, 9
}
```

**Метки для break/continue:**

```go
outer:
for i := 0; i < 10; i++ {
    for j := 0; j < 10; j++ {
        if j == 5 {
            break outer  // выйти из внешнего цикла
        }
        if i == 3 {
            continue outer  // перейти к следующей итерации внешнего
        }
    }
}
```

### for range: перебор коллекций

**`range`** — это способ перебрать элементы слайса, мапы, строки или канала.

```go
// Слайс
numbers := []int{1, 2, 3}
for index, value := range numbers {
    fmt.Println(index, value)
}

// Мапа
m := map[string]int{"a": 1, "b": 2}
for key, value := range m {
    fmt.Println(key, value)
}

// Строка (перебирает руны)
for index, rune := range "привет" {
    fmt.Println(index, rune)
}

// Канал
for value := range channel {
    fmt.Println(value)
}
```

### Когда не нужен индекс или значение

Используй `_` (blank identifier), чтобы отбросить ненужное:

```go
// Только значения, индексы не нужны
for _, value := range numbers {
    fmt.Println(value)
}

// Только индексы, значения не нужны
for index := range numbers {
    fmt.Println(index)
}
```

### Подводный камень: переменная цикла

**До Go 1.22:** переменная цикла создавалась один раз и переиспользовалась. Это приводило к багам с замыканиями.

```go
// ❌ До Go 1.22: все горутины выведут 3
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)
    }()
}

// ✅ Решение: передать как аргумент
for i := 0; i < 3; i++ {
    go func(i int) {
        fmt.Println(i)
    }(i)
}
```

**В Go 1.22+:** переменная цикла создаётся заново для каждой итерации. Старый код с передачей аргумента всё ещё работает и считается идиоматичным.

---

## 🎯 switch: выбор из вариантов

**`switch`** — это способ выбрать один вариант из нескольких. Альтернатива длинной цепочке `if-else`.

### Базовый синтаксис

```go
switch status {
case "active":
    fmt.Println("Активен")
case "inactive":
    fmt.Println("Неактивен")
default:
    fmt.Println("Неизвестно")
}
```

### Автоматический break

В Go не нужен `break` в конце `case` — он добавляется автоматически.

```go
switch x {
case 1:
    fmt.Println("Один")  // нет break, но он подразумевается
case 2:
    fmt.Println("Два")
}
```

### Несколько значений в case

```go
switch day {
case "суббота", "воскресенье":
    fmt.Println("Выходной")
default:
    fmt.Println("Рабочий")
}
```

### switch без условия

```go
switch {
case x > 10:
    fmt.Println("Больше 10")
case x > 5:
    fmt.Println("Больше 5")
default:
    fmt.Println("5 или меньше")
}
```

Это альтернатива цепочке `if-else if-else`.

### Инициализация в switch

Как и в `if`, можно объявить переменную:

```go
switch x := getValue(); x {
case 1:
    fmt.Println("Один:", x)
case 2:
    fmt.Println("Два:", x)
}
```

### Подводный камень: fallthrough

Иногда нужно выполнить следующий `case` тоже. Для этого есть `fallthrough`:

```go
switch x {
case 1:
    fmt.Println("Один")
    fallthrough  // выполнить следующий case тоже
case 2:
    fmt.Println("Два")
}
// Вывод: "Один", "Два"
```

**Но:** `fallthrough` должен быть последним оператором в `case`. И он передаёт управление в следующий `case` **без проверки условия**.

---

## ↩️ goto: переход по метке

**`goto`** — это способ перейти к метке в коде. В большинстве случаев не нужен.

```go
goto label
// ... код ...
label:
    fmt.Println("Перешли сюда")
```

### Когда goto оправдан

**1. Выход из вложенных циклов:**

```go
// ❌ Без goto: нужен флаг
found := false
for _, row := range matrix {
    for _, cell := range row {
        if cell == target {
            found = true
            break
        }
    }
    if found {
        break
    }
}

// ✅ С goto: яснее
for _, row := range matrix {
    for _, cell := range row {
        if cell == target {
            goto found
        }
    }
}
found:
    fmt.Println("Нашли")
```

**2. Обработка ошибок в одном месте:**

```go
func process() error {
    resource, err := acquire()
    if err != nil {
        goto cleanup
    }
    
    err = use(resource)
    if err != nil {
        goto cleanup
    }
    
cleanup:
    release(resource)
    return err
}
```

Но чаще лучше использовать `defer`.

### Когда goto вреден

**Спагетти-код:**

```go
// ❌ Ужасно
goto step2
step1:
// код
goto step3
step2:
// код
goto step1
step3:
// код
```

**Правило:** если `goto` делает код менее читаемым — не используй его.

---

## 🎯 Практические паттерны

### 1. Ранний возврат вместо вложенности

```go
// ❌ Плохо
func process(user *User) {
    if user != nil {
        if user.Active {
            if user.Balance > 0 {
                // код
            }
        }
    }
}

// ✅ Лучше
func process(user *User) {
    if user == nil {
        return
    }
    if !user.Active {
        return
    }
    if user.Balance <= 0 {
        return
    }
    // код
}
```

### 2. switch для типов (type switch)

```go
func describe(v interface{}) {
    switch v := v.(type) {
    case int:
        fmt.Println("Целое:", v)
    case string:
        fmt.Println("Строка:", v)
    default:
        fmt.Println("Неизвестно")
    }
}
```

### 3. for с каналом для завершения

```go
for {
    select {
    case msg := <-channel:
        process(msg)
    case <-done:
        return  // выход из цикла
    }
}
```

---

## 🧬 Внутреннее устройство: как компилятор видит управляющие конструкции

Когда компилятор обрабатывает управляющие конструкции, он преобразует их в **граф потока управления** (Control Flow Graph, CFG).

### Граф потока управления (CFG)

**CFG** — это представление программы, где:

- **Блоки** — последовательности инструкций без ветвлений
- **Рёбра** — переходы между блоками (ветвления)

```
┌─────────────────────────────────────────────────────────────┐
│  if x > 0 { ... } else { ... }                              │
│                                                             │
│  [Entry]                                                    │
│     │                                                       │
│     ▼                                                       │
│  [x > 0?] ← условие (branch)                                │
│   │    │                                                    │
│   │    └──────────────┐                                     │
│   ▼                   ▼                                     │
│  [then]            [else]  ← блоки                          │
│   │                   │                                     │
│   └────────┬──────────┘                                     │
│            ▼                                                │
│         [Exit]                                              │
└─────────────────────────────────────────────────────────────┘
```

### if/else в CFG

```go
if x > 0 {
    fmt.Println("positive")
} else {
    fmt.Println("negative")
}
```

**Компилятор преобразует в:**

```
┌─────────────────────────────────────────────────────────────┐
│  Block 1 (entry):                                           │
│    t1 = x > 0                                               │
│    branch t1 to Block 2, Block 3                            │
│                                                             │
│  Block 2 (then):                                            │
│    call fmt.Println("positive")                             │
│    jump to Block 4                                          │
│                                                             │
│  Block 3 (else):                                            │
│    call fmt.Println("negative")                             │
│    jump to Block 4                                          │
│                                                             │
│  Block 4 (exit):                                            │
│    return                                                   │
└─────────────────────────────────────────────────────────────┘
```

### for цикл в CFG

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

**Компилятор преобразует в:**

```
┌─────────────────────────────────────────────────────────────┐
│  Block 1 (init):                                            │
│    i = 0                                                    │
│    jump to Block 2                                          │
│                                                             │
│  Block 2 (cond):                                            │
│    t1 = i < 10                                              │
│    branch t1 to Block 3, Block 5                            │
│                                                             │
│  Block 3 (body):                                            │
│    call fmt.Println(i)                                      │
│    jump to Block 4                                          │
│                                                             │
│  Block 4 (incr):                                            │
│    i = i + 1                                                │
│    jump to Block 2                                          │
│                                                             │
│  Block 5 (exit):                                            │
│    return                                                   │
└─────────────────────────────────────────────────────────────┘
```

**Цикл = обратное ребро (back edge)** от Block 4 к Block 2.

### break/continue в CFG

```go
for i := 0; i < 10; i++ {
    if i == 5 {
        break
    }
    if i%2 == 0 {
        continue
    }
    fmt.Println(i)
}
```

**Компилятор преобразует в:**

```
┌─────────────────────────────────────────────────────────────┐
│  Block 1 (init): i = 0 → Block 2                            │
│  Block 2 (cond): i < 10 → Block 3, Block 7 (exit)           │
│  Block 3 (body):                                            │
│    i == 5 → Block 7 (break!)                                │
│    i%2 == 0 → Block 4 (continue!)                           │
│    Block 4 (incr): i = i + 1 → Block 2                      │
│    Block 5: println(i) → Block 4                            │
│  Block 7 (exit): return                                     │
└─────────────────────────────────────────────────────────────┘
```

**break** → переход к блоку после цикла.
**continue** → переход к блоку инкремента.

### switch в CFG

```go
switch x {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
default:
    fmt.Println("other")
}
```

**Компилятор преобразует в:**

```
┌─────────────────────────────────────────────────────────────┐
│  Block 1:                                                   │
│    branch x to Block 2 (x==1), Block 3 (x==2), Block 4      │
│                                                             │
│  Block 2 (case 1):                                          │
│    println("one")                                           │
│    jump to Block 5                                          │
│                                                             │
│  Block 3 (case 2):                                          │
│    println("two")                                           │
│    jump to Block 5                                          │
│                                                             │
│  Block 4 (default):                                         │
│    println("other")                                         │
│    jump to Block 5                                          │
│                                                             │
│  Block 5 (exit):                                            │
│    return                                                   │
└─────────────────────────────────────────────────────────────┘
```

### range в CFG

```go
for i, v := range slice {
    fmt.Println(i, v)
}
```

**Компилятор преобразует в:**

```
┌─────────────────────────────────────────────────────────────┐
│  Block 1 (init):                                            │
│    len = len(slice)                                         │
│    i = 0                                                    │
│    jump to Block 2                                          │
│                                                             │
│  Block 2 (cond):                                            │
│    t1 = i < len                                             │
│    branch t1 to Block 3, Block 5                            │
│                                                             │
│  Block 3 (body):                                            │
│    v = slice[i]                                             │
│    call println(i, v)                                       │
│    jump to Block 4                                          │
│                                                             │
│  Block 4 (incr):                                            │
│    i = i + 1                                                │
│    jump to Block 2                                          │
│                                                             │
│  Block 5 (exit):                                            │
│    return                                                   │
└─────────────────────────────────────────────────────────────┘
```

### SSA представление

После построения CFG, компилятор преобразует код в **SSA** (Static Single Assignment):

```go
// Исходный код
x := 1
if x > 0 {
    x = 2
} else {
    x = 3
}
fmt.Println(x)
```

**SSA представление:**

```
Block 1:
    x1 = 1
    t1 = x1 > 0
    branch t1 to Block 2, Block 3

Block 2 (then):
    x2 = 2
    jump to Block 4

Block 3 (else):
    x3 = 3
    jump to Block 4

Block 4 (merge):
    x4 = φ(x2, x3)  // φ-функция выбирает значение
    call println(x4)
```

**φ-функция** выбирает правильное значение в зависимости от того, по какому пути пришли.

### Оптимизации на CFG

**1. Dead Code Elimination:**

```go
if false {
    // мёртвый код → удаляется
}
```

**2. Constant Folding:**

```go
if 5 > 0 {  // всегда true → else удаляется
    // ...
} else {
    // мёртвый код
}
```

**3. Loop Invariant Code Motion:**

```go
for i := 0; i < n; i++ {
    x := expensive()  // выносится за цикл
    use(x, i)
}
```

---

## 🎯 Итог

Управляющие конструкции в Go проще, чем во многих языках, но достаточно мощные для любых задач.

**Запомни:**

- `if` — для условий, с инициализацией в условии
- `for` — единственный цикл, заменяет `while`
- `for range` — для перебора коллекций
- `break` — выход из цикла
- `continue` — пропустить итерацию
- `switch` — для выбора из вариантов, с автоматическим `break`
- `goto` — только когда делает код яснее (выход из вложенных циклов)

**Избегай:**

- Вложенных `if` (пирамида ужаса)
- `goto` без веской причины
- Сложных условий в `for`
- Меток без необходимости (код становится менее читаемым)

Понимание управляющих конструкций — основа написания чистого, читаемого кода на Go.
