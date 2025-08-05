# Сеть в Kubernetes

## Что такое Kubernetes service?

**Service** - абстракция, которая позволяет организовывать доступ к приложениям, работающим в Kubernetes, через постоянный IP-адрес или DNS-имя.
Сервис может выполнять балансировку трафика между POD'ами.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

## Каким образом можно предоставить приложение, которые работает в кластере, пользователям?

Если приложение работает по HTTP, то можно использовать `Ingress-контроллер`.
Ingress позволяет маршрутизировать HTTP/HTTPS-трафик от пользователей к приложениям в кластере.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

Если приложение работает по бинарному протоколу(PostgreSQL), то обычно используют Service с типом **NodePort** или **LoadBalancer**, чтобы сделать его доступным извне.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: NodePort
  ports:
    - port: 5432
      targetPort: 5432
      nodePort: 30000
  selector:
    app: postgres
```

### Почему Ingress удобен?

Используя Ingress, можно управлять маршрутизацией трафика для всех приложений внутри кластера через одну точку входа.
Это позволяет эффективно управлять HTTP/HTTPS-трафиком, используя такие функции как:

- Балансировка нагрузки
- Поддержка HTTPS
- Маршрутизация по поддоменам или URL
- Поддержка канареечных развертываний

## Каким образом организована сеть в k8s?

В Kubernetes существует три типа сети:

- **Node Network** - сеть, в которую объединены ноды.
В зависимости от использования CNI-плагина, ноды могут работать только в одной подсети, либо в нескольких.
- **Pod Network** - сеть, в которой получают IP-адреса запускаемые PODы.
- **Service Network** - сеть, в которой получают адреса Kubernetes services.

Pod network и service network орнанизуются при помощи так называемых CNI-плагинов.


## Что такое CNI-плагин и для чего он нужен?

CNI расшифровывается как **Container Network Interface**.
Он представляет собой некий уровень абстракции над реализацией сети.
Мы может работать с верхнеуровневыми абстракциями вроде "IP-адрес POD'а", Endpoint.
За то как это будет реализовано на физическом уровне отвечают CNI-плагины, которые реализуют разный функционал и показывают разную сетевую производительность.

Примеры CNI-плагинов:

- [Flannel](https://github.com/flannel-io/flannel)
- [Calico](https://github.com/projectcalico/calico)
- [Cilium](https://github.com/cilium/cilium)

## Что такое EGRESS?

**EGRESS** - возможность назначить внешний IP-адрес для исходящего за пределы кластера k8s трафика приложений.
Поддержка EGRESS должна быть реализована на уровне CNI-плагина и может быть описана специальным объектом на уровне неймспейса.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
```

## Как мы можем ограничить трафик в Kubernetes?

Для ограничения трафика от приложений используется объект **NetworkPolicy**.
При помощи него мы можем ограничивать входящий и исходящий трафик на уровне неймспейса и компонентов, описанных в нем.
Его поддержка должна быть реализована на уровне CNI-плагина.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
```
