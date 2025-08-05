# Безопасность в Kubernetes

## Каким образом мы можем разделить права на Kubernetes?

Для разделения прав в Kubernetes применяется механизм **Role Based Access Control(RBAC)**.
В рамках него есть три группы сущностей:

- *user* или *service account* - который описывает субъект доступа.
- *role* или *cluster role* - который описывает разрешения.
- *roleBinding* или *clusterRoleBinding* - который привязывает списки разрешений к субъекту.

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
# Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
  - kind: User
    name: example-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### В чем отличие service account от user?

**User** не имеет записей в Kubernetes API, управление осуществляется внешними механизмами.
Они предназначены для событий вне кластера.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: my-namespace
```

## Какие механизмы аутентификации используются в Kubernetes?

Kubernetes может использовать:

- сертификаты X509
- Bearer-токены
- аутентифицирующий прокси
- HTTP Basic Auth

При помощи этих механизмов можно реализовывать большое количество схем авторизации: от статичного файла с паролями до OpenID OAuth2.

Более того, допускается применение нескольких схем авторизации одновременно.
По умолчанию используются:

- service account tokens - для Service Accounts
- X509 - для Users
