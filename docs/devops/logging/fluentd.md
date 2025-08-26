---
title: Fluentd
description: Обзор Fluentd, универсального сборщика логов.
---

# Fluentd: Унифицированный сбор логов

!!! abstract "Тезис"
    **Fluentd** — это сборщик данных с открытым исходным кодом, который позволяет унифицировать сбор и обработку логов из различных источников. Его гибкая архитектура плагинов делает его "универсальным коннектором" между любым источником и любым местом назначения данных.

---

## Ключевые возможности

-   **Унифицированный сбор**: Собирает логи из файлов, syslog, TCP, HTTP и сотен других источников.
-   **Гибкая обработка**: Позволяет парсить, фильтровать, буферизировать и обогащать логи перед отправкой.
-   **Надежность**: Поддерживает буферизацию на диске или в памяти, а также механизмы повторной отправки для предотвращения потери данных.
-   **Огромная экосистема плагинов**: Сотни плагинов для интеграции с различными источниками, базами данных, системами мониторинга и облачными сервисами.

---

## Архитектура

!!! example "Схема работы Fluentd"
    ```plaintext
    +----------+     +-----------------+     +------------------+
    |  Source  | --> | Fluentd Agent   | --> | Destination      |
    | (e.g.    |     | +-------------+ |     | (e.g. Elasticsearch,
    | nginx log|     | | Input Plugin| |     |  S3, Kafka)      |
    | file)    |     | +-------------+ |     +------------------+
    +----------+     | | Filter Plugin||
                   | +-------------+ |
                   | | Output Plugin||
                   | +-------------+ |
                   +-----------------+
    ```

-   **Input Plugin**: Определяет источник данных (e.g., `in_tail` для файлов, `in_forward` для TCP).
-   **Filter Plugin**: Модифицирует логи (e.g., `filter_grep` для фильтрации, `filter_parser` для парсинга).
-   **Output Plugin**: Отправляет данные в место назначения (e.g., `out_elasticsearch`, `out_s3`).
-   **Buffer**: Временно хранит данные перед отправкой для обеспечения надежности.

---

## Пример конфигурации

!!! example "Пример `fluent.conf`"
    ```aconf
    # Source: tail a file
    <source>
      @type tail
      path /var/log/nginx/access.log
      pos_file /var/log/td-agent/nginx.pos
      tag nginx.access
      <parse>
        @type nginx
      </parse>
    </source>

    # Filter: add hostname to records
    <filter nginx.access>
      @type record_transformer
      <record>
        hostname ${hostname}
      </record>
    </filter>

    # Destination: send to Elasticsearch
    <match nginx.access>
      @type elasticsearch
      host elasticsearch
      port 9200
      logstash_format true
      logstash_prefix fluentd
      <buffer>
        @type file
        path /var/log/td-agent/buffer/es
        flush_interval 10s
      </buffer>
    </match>
    ```

---

## Fluentd vs Fluent Bit

**Fluent Bit** — это легковесный форвардер логов от тех же создателей. Он написан на C (Fluentd на Ruby) и оптимизирован для высокой производительности и низкого потребления ресурсов.

| Критерий | Fluentd | Fluent Bit |
| :--- | :--- | :--- |
| **Производительность** | Хорошая | Очень высокая |
| **Потребление ресурсов**| Среднее | Низкое |
| **Плагины** | Огромное количество | Меньше, но ключевые есть |
| **Use Case** | Центральный агрегатор | Сборщик на edge-устройствах, sidecar в Kubernetes |

!!! success "Рекомендация"
    Часто используется гибридный подход: **Fluent Bit** работает на узлах как легковесный сборщик, а затем пересылает логи на центральный агрегатор **Fluentd** для сложной обработки и маршрутизации.
