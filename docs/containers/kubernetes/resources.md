# Управление ресурсами

## Каким образом можно управлять вычислительными ресурсами в k8s?

**Resources requests** и **limits** позволяют управлять количеством ресурсов, которое контейнер может использовать.

- **Request** - минимальное количество ресурсов, которое контейнер гарантировано получит.
- **Limits** - максимальное количество ресурсов, которое контейнер может использовать.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:v1
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
