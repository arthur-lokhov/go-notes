# Хранилище (Storage)

## Какие типы volum'ов можно использовать в Kubernetes?

1. hostpath, но тогда POD должен быть привязан к ноде.
2. local-storage - автоматически привязывает POD, который его использует, к нужной ноже.

Также можно использовать сетевые диски при помощи CSI-плагинов.

## Что такое CSI-плагин?

CSI расшифровывается как **Container Storage Interface**.
Это абстракция, позволяющая унифицированно использовать сетевые файловые системы, построенные на разных технологических базах.

Мы описываем *storageClass*, соответствующий дискам определенного типа, и деплоим в кластер *provisioner*.

**Provisioner** - специальное ПО, которое может заказывать сетевые диски в системе, способной их предоставлять.

Далее мы описываем объект *persistentVolumeClaim* с указанием нужного *storageClass*.

Provisioner при появлении *PVC* заказывает диск нужного размера в системе, которая их предоставляет, создает объект *persistentVolume* и привязывает его к *PVC*.
Когда происходит запуск POD'а на ноде, соотвествующий диск монтируется на нужную ноду по определенному пути, и этот путь монтируется на файловую систему PODа.

```yaml
# Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2

---
# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast
```
