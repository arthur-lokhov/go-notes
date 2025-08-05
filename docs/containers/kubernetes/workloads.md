# Рабочие нагрузки (Workloads)

## Что такое POD?

**Pod** - минимальная единица развертывания в Kubernetes, которая предоставляет собой группу контейнеров, объединенных общей сетью(общий localhost, общий внешний IP) и общими ресурсами.
Контейнеры внутри POD работают в тесной связке.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: php-fpm
    image: php:fpm
```

### Для чего при старте PODа создается контейнер с процессом pause?

Для того чтобы создать сетевую изоляцию Kubernetes при старте POD'а запускает специальный контейнер `pause`.
Он отвечает за управление сетью и обеспечивает контейнерам внутри POD'а возможность взаимодействовать через localhost.
Он не выполняет реальную работу, а является посредником при сетевом взаимодействии.

### Как можно запускать поды?

POD'ы обычно запускаются с помощью **workloads**, которые обеспечивают управление жизненным циклом POD'ов.

Примеры:

1. ***(ReplicationController)*** **Deployment** - для развертывания и управления репликами POD'ов с возможность откатов.
2. **StatefulSet** - для приложений с состоянием, которые требуют уникальности и постоянства идентификаторов.
3. **DaemonSet** - для запуска одного POD'а на каждом узле кластера.
4. **Job** - для запуска одноразовых задач.
5. **CronJob** - для запуска задач по расписанию.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```

## В чем разница между Deployment и StatefulSet?

1. В именах: у первого некий случайный хэш, у второго - порядковый номер.
2. В работе с дисками: StatefulSet дожидается пока POD с таким же именем завершит работы, чтобы занять тот диск, который был к нему привязан.
3. В стратегиях перезапуска POD'ов при обновлении.

## Что такое ReplicaSet?

**ReplicaSet** - это объект, который управляет количеством реплик POD'ов в Kubernetes.
ReplicaSet гарантирует, что в любой момент времени будет существовать определенное количество копий POD'ов.
В Kubernetes обычно используют Deployment, который управляет ReplicaSet.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```
