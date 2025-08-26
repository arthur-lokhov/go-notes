---
title: Polkit (PolicyKit)
---

# 🛡️ Polkit: Управление привилегиями в Userspace

Если **PAM** — это "фейс-контроль" на входе в систему, то **Polkit (PolicyKit)** — это служба безопасности внутри, которая решает, можно ли пользователю выполнить конкретное привилегированное действие, уже находясь в системе. Polkit отвечает на вопрос: "*Можно ли пользователю `david` перезапустить сервис `nginx` из графического интерфейса?*"

## 🎯 Цель раздела

- Понять архитектуру Polkit и его роль в современных десктопных и серверных окружениях.
- Разобраться в ключевых понятиях: **действие (action)**, **субъект (subject)**, **авторизация (authorization)**.
- Научиться читать и писать правила Polkit для гранулярного контроля над системными действиями.
- Сравнить Polkit и `sudo` и понять, когда использовать каждый из них.

---

## 🏛️ Архитектура Polkit

Polkit — это централизованный сервис авторизации, который работает поверх **D-Bus**. Когда приложение (субъект) хочет выполнить привилегированное действие, оно не вызывает `sudo`, а отправляет запрос демону Polkit.

**Схема взаимодействия:**

```
+------------------+
| Приложение (GUI) |
| (GNOME Settings) |  <-- 1. Пользователь нажимает "Перезапустить сеть"
+------------------+
         ↓ (D-Bus call: org.freedesktop.NetworkManager.Restart)
+------------------+
|   D-Bus брокер   |
+------------------+
         ↓
+------------------+
|   Polkit демон   |  <-- 2. Polkit перехватывает вызов и ищет действие
|    (polkitd)     |      (org.freedesktop.NetworkManager.settings.modify.system)
+------------------+
         ↓
+------------------------------------+
| Правила Polkit                       |  <-- 3. Polkit проверяет правила в:
| (/usr/share/polkit-1/actions/*.policy) |
| (/etc/polkit-1/rules.d/*.rules)      |
+------------------------------------+
         ↓ (Результат: ДА/НЕТ/ПАРОЛЬ)
+------------------+
|   Polkit агент   |  <-- 4. Если нужен пароль, агент (GUI-диалог)
| (GUI-диалог)     |      запрашивает его у пользователя.
+------------------+
         ↓
+------------------+
| Системный сервис |
| (NetworkManager) |  <-- 5. Если авторизация пройдена, действие выполняется.
+------------------+
```

- **Субъект (Subject):** Процесс, который инициирует действие (например, `gnome-control-center`).
- **Действие (Action):** Привилегированная операция, которую хочет выполнить субъект. Действия имеют уникальные имена в формате `com.vendor.application.action` (например, `org.freedesktop.login1.reboot`). Они описываются в `.policy` файлах.
- **Демон Polkit (`polkitd`):** Центральный процесс, который принимает запросы, проверяет правила и выносит решение.
- **Агент аутентификации (Authentication Agent):** Графический или текстовый диалог, который запрашивает пароль у пользователя, если это требуется правилами.
- **Правила (Rules):** Файлы, которые определяют, при каких условиях действие разрешено. Современный формат — JavaScript-файлы (`.rules`).

---

## ⚖️ Polkit vs. `sudo`

| Характеристика | 🛡️ Polkit | 🔑 `sudo` |
| :--- | :--- | :--- |
| **Гранулярность** | **Высокая.** Контролирует конкретные действия (API вызовы). "Разрешить монтировать, но не форматировать". | **Низкая.** Даёт полный доступ к выполнению команды от имени другого пользователя (обычно `root`). | 
| **Интерактивность** | **Ориентирован на GUI.** Легко интегрируется с десктопом, показывает красивые диалоги. | **Ориентирован на CLI.** Основной инструмент для командной строки. |
| **Конфигурация** | JavaScript-правила или `.policy` файлы. Более сложная, но гибкая. | Простой текстовый файл `/etc/sudoers`. |
| **Сценарий** | Управление системными сервисами из GUI, монтирование дисков, управление питанием. | Выполнение административных команд в терминале, запуск скриптов с правами `root`. |

!!! success "Вывод"
    **Polkit не заменяет `sudo`.** Они решают разные задачи. `sudo` — для интерактивной работы в терминале. Polkit — для программного, гранулярного контроля над действиями в системе, особенно в графическом окружении.

---

## 📜 Правила Polkit

Современный способ определения политик — это создание JavaScript-файлов в `/etc/polkit-1/rules.d/`.

### Чтение действий

Чтобы понять, какие действия доступны в системе, можно посмотреть `.policy` файлы в `/usr/share/polkit-1/actions/`.

**Пример (`org.freedesktop.login1.policy`):**
```xml
<action id="org.freedesktop.login1.reboot">
  <description>Reboot the system</description>
  <message>Authentication is required to reboot the system.</message>
  <defaults>
    <allow_any>auth_admin_keep</allow_any>
    <allow_inactive>auth_admin_keep</allow_inactive>
    <allow_active>yes</allow_active>
  </defaults>
</action>
```
- **`allow_any`:** Для неактивных сессий (например, по SSH). `auth_admin_keep` — запросить пароль администратора и запомнить авторизацию на некоторое время.
- **`allow_inactive`:** То же самое.
- **`allow_active`:** Для активной локальной сессии (за компьютером). `yes` — разрешить без пароля.

Вот почему пользователь за своим ноутбуком может перезагрузить его без пароля, а при подключении по SSH — требуется пароль `sudo`.

### Написание правил

**Задача:** Разрешить пользователям из группы `operators` перезапускать `nginx` без пароля.

1.  **Находим имя действия.** Обычно оно связано с D-Bus интерфейсом сервиса. Для `systemd`-сервисов это `org.freedesktop.systemd1.manage-units`.

2.  **Создаём файл `/etc/polkit-1/rules.d/10-nginx-restart.rules`:**
    ```javascript
    // Разрешить пользователям группы 'operators' управлять сервисом nginx
    polkit.addRule(function(action, subject) {
        // Проверяем, что действие - это управление юнитами systemd
        if (action.id == "org.freedesktop.systemd1.manage-units") {
            // Проверяем, что юнит, которым управляют - это nginx.service
            if (action.lookup("unit") == "nginx.service") {
                // Проверяем, что пользователь состоит в группе 'operators'
                if (subject.isInGroup("operators")) {
                    // Разрешаем действие
                    return polkit.Result.YES;
                }
            }
        }
    });
    ```

**Разбор:**
- `polkit.addRule(...)`: Регистрирует новое правило.
- `function(action, subject)`: Функция, которая вызывается для каждой проверки. `action` — объект с информацией о действии, `subject` — о пользователе.
- `action.id`: Идентификатор действия.
- `action.lookup("unit")`: Polkit позволяет передавать параметры вместе с действием. Здесь мы проверяем параметр `unit`.
- `subject.isInGroup("operators")`: Проверяет членство в группе.
- `polkit.Result.YES`: Результат — разрешить. Другие варианты: `polkit.Result.NO` (запретить), `polkit.Result.AUTH_ADMIN` (запросить пароль админа).

---

### Go Example: Проверка авторизации через D-Bus

Этот пример на Go показывает, как приложение может программно проверить, авторизован ли текущий пользователь для выполнения действия, используя D-Bus интерфейс Polkit.

```go
package main

import (
	"fmt"
	"os"

	"github.com/godbus/dbus/v5"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage: go run main.go <action_id>")
		fmt.Println("Example: go run main.go org.freedesktop.login1.reboot")
		os.Exit(1)
	}

	actionID := os.Args[1]

	// Подключаемся к системной шине D-Bus
	conn, err := dbus.ConnectSystemBus()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Failed to connect to system bus: %v\n", err)
		os.Exit(1)
	}
	defer conn.Close()

	// Создаём объект для вызова методов Polkit
	obj := conn.Object("org.freedesktop.PolicyKit1", "/org/freedesktop/PolicyKit1/Authority")

	// Описываем субъект (текущий процесс)
	subject := map[string]dbus.Variant{
		"pid": dbus.MakeVariant(uint32(os.Getpid())),
	}

	// Флаги для проверки (0 - стандартное поведение)
	flags := uint32(0)

	// Детали действия (пусто, если не нужны)
	details := map[string]string{}

	var result []interface{}

	// Вызываем метод CheckAuthorization
	err = obj.Call(
		"org.freedesktop.PolicyKit1.Authority.CheckAuthorization",
		0,
		subject,
		actionID,
		details,
		flags,
		"", // cancellation_id
	).Store(&result)

	if err != nil {
		fmt.Fprintf(os.Stderr, "Failed to call CheckAuthorization: %v\n", err)
		os.Exit(1)
	}

	// Разбираем результат
	isAuthorized := result[0].(bool)
	isChallenge := result[1].(bool)
	authDetails := result[2].(map[string]string)

	fmt.Printf("Action: %s\n", actionID)
	fmt.Printf("Is Authorized: %t\n", isAuthorized)
	fmt.Printf("Is Challenge (needs password): %t\n", isChallenge)
	fmt.Printf("Details: %v\n", authDetails)

	if isAuthorized {
		fmt.Println("\nPermission granted!")
	} else {
		fmt.Println("\nPermission denied.")
	}
}
```
