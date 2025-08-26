---
title: Шифрование дисков
---

# 🔐 Шифрование дисков: LUKS и cryptsetup

Шифрование данных в состоянии покоя (at-rest) — критически важный компонент безопасности. Оно защищает конфиденциальную информацию в случае физической кражи или несанкционированного доступа к носителю. В Linux стандартом де-факто для шифрования блочных устройств является **LUKS (Linux Unified Key Setup)**.

## 🎯 Цель раздела

- Понять, что такое LUKS и как он работает.
- Разграничить подходы: **полное шифрование диска (FDE)** и **шифрование контейнеров**.
- Научиться создавать и управлять зашифрованными томами с помощью `cryptsetup`.
- Изучить автоматическую разблокировку при загрузке через `/etc/crypttab` и ключевые файлы.

---

## 🛡️ Что такое LUKS?

**LUKS** — это не само шифрование, а **стандарт спецификации для управления шифрованием**. Он предоставляет надёжный и гибкий слой поверх стандартного механизма ядра Linux — **dm-crypt**.

Ключевые особенности LUKS:

- **Заголовок с метаданными:** LUKS хранит всю необходимую информацию (тип шифра, хэш, и т.д.) в специальном заголовке на самом диске. Это делает зашифрованный том самодостаточным и переносимым.
- **Поддержка нескольких ключей:** Вы можете зашифровать том одним мастер-ключом, но предоставить к нему доступ с помощью нескольких разных паролей или ключевых файлов. Это позволяет легко отзывать доступ (удаляя один из слотов), не перешифровывая весь диск.
- **Защита от брутфорса:** LUKS использует PBKDF2 (Password-Based Key Derivation Function 2), что делает подбор пароля чрезвычайно медленным и ресурсоёмким.

**Как это работает:**

1.  **Мастер-ключ (Master Key):** При создании LUKS-тома генерируется один основной, очень сильный ключ шифрования. Именно им шифруются все данные.
2.  **Слоты для ключей (Key Slots):** Мастер-ключ сам шифруется и помещается в один из 8 доступных слотов. То, чем он шифруется — это ваш пароль (пропущенный через PBKDF2) или ключевой файл.
3.  **Разблокировка:** Когда вы вводите пароль, LUKS пытается расшифровать им каждый слот. Если успешно — он получает доступ к мастер-ключу, загружает его в ядро (dm-crypt) и открывает зашифрованный том.

![LUKS Structure](https://i.imgur.com/example.png) *<-- Placeholder for a real diagram of LUKS key slots and master key.*

---

## 📦 Полное шифрование (FDE) vs. Контейнеры

| Подход | 💿 Полное шифрование диска (FDE) | 🗃️ Файл-контейнер |
| :--- | :--- | :--- |
| **Что шифруется** | Весь раздел или диск (например, `/dev/sda2`). Часто используется для корневой ФС или `/home`. | Обычный файл, который монтируется как блочное устройство. | 
| **Настройка** | Выполняется при установке ОС или на пустом разделе. | Можно создать в любое время в домашнем каталоге. |
| **Гибкость** | Низкая. Размер фиксирован размером раздела. | Высокая. Файл можно легко перемещать, копировать, бэкапить. |
| **Сценарий** | Защита всей операционной системы на ноутбуке или сервере. | Хранение набора конфиденциальных файлов, переносных секретов. |

---

## 🛠️ Управление LUKS с `cryptsetup`

`cryptsetup` — основной инструмент для работы с LUKS и dm-crypt.

### Создание зашифрованного раздела

!!! warning "Внимание!"
    Следующая команда **уничтожит все данные** на `/dev/sdb1`.

```bash
# Создаём LUKS-том на разделе /dev/sdb1
# cryptsetup luksFormat /dev/sdb1

WARNING! This will overwrite data on /dev/sdb1 irrevocably.
Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sdb1: 
Verify passphrase: 
```

### Открытие и отображение тома

Чтобы начать работать с томом, его нужно открыть. `cryptsetup` создаст "виртуальное" блочное устройство в `/dev/mapper/`.

```bash
# Открываем том и даём ему имя "encrypted_data"
# cryptsetup open /dev/sdb1 encrypted_data
Enter passphrase for /dev/sdb1: 

# Проверяем
# ls -l /dev/mapper/
crw-rw---- 1 root root 10, 236 Jul 28 10:00 control
lrwxrwxrwx 1 root root       7 Jul 28 10:10 encrypted_data -> ../dm-0
```

Теперь `/dev/mapper/encrypted_data` — это обычное блочное устройство, на котором можно создать ФС.

```bash
# mkfs.ext4 /dev/mapper/encrypted_data
# mkdir /secure_data
# mount /dev/mapper/encrypted_data /secure_data
```

### Закрытие тома

```bash
# umount /secure_data
# cryptsetup close encrypted_data
```

---

## 🔑 Управление ключами

### Добавление нового пароля

Вы можете добавить второй пароль, не зная первого (но имея доступ к одному из существующих).

```bash
# cryptsetup luksAddKey /dev/sdb1
Enter any existing passphrase: <старый_пароль>
Enter new passphrase for key slot: <новый_пароль>
Verify passphrase: 
```

### Использование ключевого файла (Keyfile)

Пароли неудобны для автоматической разблокировки. Для этого используют ключевые файлы.

1.  **Создаём случайный ключевой файл:**
    ```bash
    # dd if=/dev/urandom of=/etc/keys/data.key bs=4096 count=1
    # chmod 0400 /etc/keys/data.key # Защищаем!
    ```

2.  **Добавляем его в свободный слот:**
    ```bash
    # cryptsetup luksAddKey /dev/sdb1 /etc/keys/data.key
Enter any existing passphrase: <ваш_пароль>
    ```

3.  **Удаляем пароль (опционально, для повышения безопасности):**
    ```bash
    # cryptsetup luksRemoveKey /dev/sdb1
Enter passphrase to be removed: <ваш_пароль>
    ```

---

## 🚀 Автоматическая разблокировка: `/etc/crypttab`

Файл `/etc/crypttab` работает аналогично `/etc/fstab` и позволяет `systemd` автоматически разблокировать устройства при загрузке.

**Формат:**
`<name> <device> <keyfile> <options>`

| Поле | Описание |
| :--- | :--- |
| **`<name>`** | Имя, которое будет присвоено устройству в `/dev/mapper/`. |
| **`<device>`** | Путь к зашифрованному устройству. **Используйте `UUID=`**. |
| **`<keyfile>`** | Путь к ключевому файлу. `none` — если нужно вводить пароль вручную. |
| **`<options>`** | Опции. `luks` — самая частая. `tries=0` — не пытаться разблокировать несколько раз. `discard` — для SSD. |

**Пример:**

1.  **Находим UUID:**
    ```bash
    # blkid /dev/sdb1
    /dev/sdb1: UUID="abcdef-1234-..." TYPE="crypto_LUKS"
    ```

2.  **Добавляем запись в `/etc/crypttab`:**
    ```
    # <target name> <source device>         <key file>      <options>
    encrypted_data  UUID=abcdef-1234-...    /etc/keys/data.key  luks
    ```

3.  **Добавляем запись в `/etc/fstab` для монтирования:**
    ```
    /dev/mapper/encrypted_data  /secure_data  ext4  defaults,nofail 0 2
    ```

4.  **Обновляем initramfs:**
    ```bash
    # update-initramfs -u -k all
    ```

Теперь при загрузке `systemd` прочитает `crypttab`, найдёт ключевой файл, разблокирует том, а затем `fstab` его смонтирует.

---

### Go Example: Проверка статуса LUKS-тома

Программа на Go, которая использует `cryptsetup` для проверки, является ли том активным и какие у него параметры.

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"os"
	"os/exec"
)

// LuksStatus представляет JSON-вывод команды `cryptsetup status --json`
type LuksStatus struct {
	Cipher    string `json:"cipher"`
	Keysize   int    `json:"keysize"`
	Device    string `json:"device"`
	Type      string `json:"type"`
	UUID      string `json:"uuid"`
}

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage: go run main.go <mapper_name>")
		fmt.Println("Example: go run main.go encrypted_data")
		os.Exit(1)
	}

	mapperName := os.Args[1]

	cmd := exec.Command("cryptsetup", "status", mapperName, "--json")

	var out bytes.Buffer
	cmd.Stdout = &out
	cmd.Stderr = os.Stderr

	err := cmd.Run()
	if err != nil {
		fmt.Fprintf(os.Stderr, "Failed to get status for %s. Is it open?\n", mapperName)
		os.Exit(1)
	}

	var status LuksStatus
	if err := json.Unmarshal(out.Bytes(), &status); err != nil {
		fmt.Fprintf(os.Stderr, "Failed to parse JSON: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Status for '%s':\n", mapperName)
	fmt.Printf("  Type:     %s\n", status.Type)
	fmt.Printf("  Device:   %s\n", status.Device)
	fmt.Printf("  UUID:     %s\n", status.UUID)
	fmt.Printf("  Cipher:   %s\n", status.Cipher)
	fmt.Printf("  Keysize:  %d bits\n", status.Keysize)
}
```
