---
title: Alertmanager
description: Обзор Alertmanager, компонента Prometheus для управления оповещениями.
---

# Alertmanager: Управление оповещениями

!!! abstract "Тезис"
    **Alertmanager** — это компонент стека Prometheus, отвечающий за обработку оповещений (alerts), отправленных сервером Prometheus. Его основная задача — дедупликация, группировка, и маршрутизация оповещений в правильные места назначения, такие как email, PagerDuty, Slack или OpsGenie.

---

## Ключевые возможности

-   **Группировка**: Объединяет оповещения одного типа в единое уведомление (например, "10 баз данных недоступны" вместо 10 отдельных сообщений).
-   **Подавление (Inhibition)**: Подавляет оповещения одного типа, если уже активно другое, более важное оповещение (например, не слать оповещения о недоступности приложения, если уже есть оповещение о недоступности всего дата-центра).
-   **Тишина (Silences)**: Позволяет временно отключать оповещения для определенных событий (например, на время планового обслуживания).
-   **Маршрутизация**: Отправляет оповещения разным командам или в разные каналы в зависимости от их лейблов.

---

## Архитектура

Alertmanager работает как отдельный сервис, который принимает оповещения от одного или нескольких серверов Prometheus.

!!! example "Схема работы Alertmanager"
    ```plaintext
    +-----------------+     (Fires alerts)     +-----------------+
    |   Prometheus    | --------------------> |  Alertmanager   |
    | (Alerting Rules)|                        | (Grouping,      |
    +-----------------+                        |  Inhibition,    |
                                             |  Routing)       |
                                             +-------+---------+
                                                     |
                         +---------------------------+--------------------------+
                         | (Notifications)           | (Notifications)          | (Notifications)
                   +-----v-----------+     +---------v----------+     +---------v----------+
                   |      Slack      |     |     PagerDuty      |     |       Email        |
                   +-----------------+     +--------------------+     +--------------------+
    ```

---

## Пример конфигурации

!!! example "Пример `alertmanager.yml`"
    ```yaml
    route:
      group_by: ['alertname', 'cluster']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 3h
      receiver: 'slack-default'

      routes:
      - match:
          severity: 'critical'
        receiver: 'pagerduty'

    receivers:
    - name: 'slack-default'
      slack_configs:
      - api_url: 'https://hooks.slack.com/services/...'
        channel: '#alerts'

    - name: 'pagerduty'
      pagerduty_configs:
      - service_key: <your_service_key>
    ```
    Эта конфигурация отправляет все оповещения по умолчанию в Slack, а оповещения с лейблом `severity: critical` — в PagerDuty.

---

!!! success "Рекомендация"
    Всегда развертывайте **Alertmanager** в отказоустойчивой конфигурации (несколько реплик), чтобы не пропустить важные оповещения.
