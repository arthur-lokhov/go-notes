---
title: Интеграция с LDAP
---

# 🌐 Интеграция с LDAP

**LDAP (Lightweight Directory Access Protocol)** — это протокол для доступа к службам каталогов. В контексте Linux, LDAP чаще всего используется для централизованного управления учётными записями пользователей, групп и данных для аутентификации. Интеграция с LDAP позволяет пользователям входить на множество серверов с единым логином и паролем.

## 🎯 Цель раздела

- Понять основы LDAP: древовидная структура (DIT), Distinguished Names (DN), атрибуты.
- Изучить, как настроить Linux-клиент для аутентификации через LDAP с помощью `sssd`.
- Разобраться в конфигурационных файлах `sssd.conf`.
- Освоить утилиты для запросов к LDAP-серверу: `ldapsearch`.

---

## 🏛️ Архитектура и концепции LDAP

LDAP организует данные в иерархическую, древовидную структуру, называемую **Directory Information Tree (DIT)**.

- **Entry (Запись):** Каждая запись в дереве представляет собой объект (например, пользователя, группу, компьютер).
- **Distinguished Name (DN):** Уникальный идентификатор каждой записи. Он формируется из пути от корня дерева до записи.
  - `uid=jdoe,ou=People,dc=example,dc=com`
- **Attributes (Атрибуты):** Каждая запись состоит из набора атрибутов, которые описывают её.
  - `uid`: `jdoe`
  - `cn` (Common Name): `John Doe`
  - `objectClass`: Определяет тип объекта (`posixAccount`, `inetOrgPerson`).
  - `homeDirectory`: `/home/jdoe`

---

## 🛡️ SSSD: System Security Services Daemon

`sssd` — это современный, предпочтительный способ интеграции Linux с удалёнными поставщиками учётных данных (LDAP, Active Directory, Kerberos).

**Преимущества `sssd`:**

- **Кэширование:** `sssd` кэширует учётные данные локально. Это позволяет пользователям входить в систему, даже если LDAP-сервер временно недоступен.
- **Отказоустойчивость:** Может работать с несколькими LDAP-серверами.
- **Производительность:** Уменьшает количество запросов к серверу.
- **Простота конфигурации:** Единый конфигурационный файл `sssd.conf`.

**Схема работы:**

1.  Приложение (например, `sshd`) вызывает PAM.
2.  PAM-модуль `pam_sss.so` передаёт запрос `sssd`.
3.  `sssd` проверяет свой кэш. Если данных нет, он делает запрос к LDAP-серверу.
4.  Получив данные, `sssd` кэширует их и возвращает результат приложению.

---

## 🛠️ Настройка LDAP-клиента с `sssd`

### 1. Установка пакетов

```bash
# Debian/Ubuntu
apt install sssd-ldap sssd-tools libnss-sss libpam-sss

# RHEL/CentOS
dnf install sssd sssd-client
```

### 2. Конфигурация `/etc/sssd/sssd.conf`

Это основной файл конфигурации. Он должен иметь права `0600`.

```ini
[sssd]
config_file_version = 2
services = nss, pam
domains = my_ldap_domain

[nss]
filter_groups = root
filter_users = root

[pam]

[domain/my_ldap_domain]
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap

# URI LDAP-сервера (можно указать несколько)
ldap_uri = ldap://ldap.example.com

# Базовый DN для поиска
ldap_search_base = dc=example,dc=com

# Включить кэширование
cache_credentials = True

# Использовать для поиска пользователей и групп
ldap_schema = rfc2307bis

# Атрибуты для пользователей и групп
ldap_user_object_class = posixAccount
ldap_user_uid_number = uidNumber
ldap_user_gid_number = gidNumber
ldap_user_home_directory = homeDirectory
ldap_user_shell = loginShell

ldap_group_object_class = posixGroup
ldap_group_gid_number = gidNumber
```

### 3. Настройка PAM и NSS

Нужно указать системе использовать `sss` для поиска пользователей и для аутентификации.

- **`/etc/nsswitch.conf`:**
  ```
  passwd:   files sss
  group:    files sss
  shadow:   files sss
  ```

- **`/etc/pam.d/common-auth` (или `system-auth`):**
  Убедитесь, что `pam_sss.so` присутствует в стеках `auth`, `account`, `password`, `session`.

### 4. Запуск и проверка

```bash
# Устанавливаем права на конфиг
chmod 600 /etc/sssd/sssd.conf

# Запускаем и включаем сервис
systemctl enable sssd --now

# Проверяем, что sssd видит LDAP-пользователя
id ldapuser
# uid=1001(ldapuser) gid=1001(ldapgroup) groups=1001(ldapgroup)

# Очистка кэша (при отладке)
sss_cache -E
```

---

## 🔬 `ldapsearch`: Отладка и запросы

`ldapsearch` — это утилита для выполнения прямых запросов к LDAP-серверу. Она незаменима для проверки, видит ли клиент сервер и какие данные он возвращает.

```bash
# Простой запрос для поиска пользователя 'jdoe'
# -x: простая аутентификация (не SASL)
# -H: URI сервера
# -b: база для поиска
# -D: DN для бинда (если требуется)
# -W: запросить пароль для бинда
ldapsearch -x -H ldap://ldap.example.com -b "dc=example,dc=com" "(uid=jdoe)"
```

Этот запрос вернёт полную LDAP-запись для пользователя `jdoe`, показывая все его атрибуты. Это помогает убедиться, что атрибуты, указанные в `sssd.conf` (`uidNumber`, `homeDirectory` и т.д.), существуют и корректны.
