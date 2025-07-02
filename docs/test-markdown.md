---
title: Full markdown specification for `mkdocs`
hide:
    - navigation
---
# 📘 Full markdown specification for `mkdocs`

## 🧱 Headers

```markdown
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

---

## 📷 Images

```markdown
![Image description](assets/logo.png){ width = 100 }

<img src="assets/logo.png" alt="Logo" width="200">
```

![Image description](assets/logo.png){ width=100 }

<img src="../assets/logo.png" alt="Logo" width="200">

---

## 🔗 Links

```markdown
[Base link](https://golang.org)

<https://golang.org>
```
[Base link](https://golang.org)

<https://golang.org>

---

## 📑 Lists

### Mark

```markdown
- Element 1
    - Inner
        - Deeper
```

- Element 1
    - Inner
        - Deeper

### Unmark

```markdown
1. First
2. Second
    1. Inner
        1. Deeper
            1. More Deeper
```

1. First
2. Second
    1. Inner
        1. Deeper
            1. More Deeper

### List of definitions

```markdown
Golang
: Programming language
: With compiler
```

Golang
: Programming language
: With compiler

---

## ✅ Task list

```markdown
- [x] Ready
- [ ] Not ready
```

- [x] Ready
- [ ] Not ready

---

## 🔤 Tables

```markdown
| Language | Status   |
|----------|----------|
| Go       | ✅       |
| Python   | ✅       |
| Java     | ⚠️       |
```

| Language | Status   |
|----------|----------|
| Go       | ✅       |
| Python   | ✅       |
| Java     | ⚠️       |

---

## 💡Inserting code

```markdown
Use `fmt.Println()` for output.
```

Use `fmt.Println()` for output.

### With `content.code.copy` and `content.code.annotate`

```markdown
```go hl_lines="6"
package main

import "fmt"

func main() {
    fmt.Println("Example with annotate")
}

```

```go hl_lines="6"
package main

import "fmt"

func main() {
    fmt.Println("Example with annotate")
}
```

---

## 📌 Snippets

```markdown
--8<-- "docs/snippets/test-snippet.md <!--Add " in the end-->

<!--test-snippet.md-->
### Test snippet

Test snippet
```

--8<-- "docs/snippets/test-snippet.md"

---

## 🪄 Tabs

=== "Go"

    ```go
    fmt.Println("Hello Go")
    ```

=== "Python"

    ```py
    print("Hello Python")
    ```

---

## 🧠 Footnote

```markdown
Footnote[^1]

[^1]: Text of footnote
```

Footnote[^1]

[^1]: Text of footnote

---

## 🛑 `admonition` blocks

```markdown
!!! note "Note"
    Note

!!! tip "Tip"
    Tip

!!! info "Info"
    Info

!!! warning "Warning"
    Warning

!!! danger "Danger"
    Danger

!!! example "Example"
    Example
```

!!! note "Note"
    Note

!!! tip "Tip"
    Tip

!!! info "Info"
    Info

!!! warning "Warning"
    Warning

!!! danger "Danger"
    Danger

!!! example "Example"
    Example

### With details

```markdown
??? note "Note with details"
    Note with details

??? tip "Tip with details"
    Tip with details

??? info "Info with details"
    Info with details

??? warning "Warning with details"
    Warning with details

??? danger "Danger with details"
    Danger with details

??? example "Example with details"
    Example with details

```

??? note "Note with details"
    Note with details

??? tip "Tip with details"
    Tip with details

??? info "Info with details"
    Info with details

??? warning "Warning with details"
    Warning with details

??? danger "Danger with details"
    Danger with details

??? example "Example with details"
    Example with details

---

## 🧱 `pymdownx` support

### `mark`, `tilde`, `caret`, `critic`

```markdown
==Mark==, ~~Tilde~~, ^caret^, {++add critic++}, {--delete critic--}
```

==Mark==, ~~Tilde~~, ^caret^, {++add critic++}, {--delete critic--}

### Inline highlight

```markdown
`#!go import fmt`
```

`#!go import fmt`

### Magic link

Use only link and this should work.

```markdown
https://github.com/arthur-lokhov/go-notes
```

https://github.com/arthur-lokhov/go-note

### Use emoji

```markdown
:smile:
```

:smile:

---

## 📎 YAML Frontmatter

```markdown
---
title: Пример
tags: [golang, mkdocs]
description: YAML-заголовок в начале документа
---
```
