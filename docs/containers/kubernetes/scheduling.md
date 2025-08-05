# Планирование (Scheduling)

## Каким образом можно управлять размещением PODов на конкретных нодах кластера k8s?

1. **NodeSelector/Node Affinity** - позволяет указать, на каких узлах можно запускать POD'ы, основываясь на метках нод. 
2. **Taints/Tolerations** - механизм, который запрещает или разрешает запуск POD'ов на определенных узлах в зависимости от их меток. 
3. **Pod Affinity/Anti-Affinity** - механизм, позволяющий задавать правила для размещения POD'ов на одних или разных узлах в зависимости от меток.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "disktype"
            operator: In
            values:
            - ssd
  containers:
  - name: myapp
    image: myapp:v1
```

## Что такое Pod Disruption Budget?

**Pod Disruption Budget (PDB)** - механизм, который гарантирует, что в кластере всегда будет доступно минимальное количество POD'ов для определенного приложения.
PDB помогает предотвратить слишком большое количество перерывов в работе приложения при обслуживании или удалении POD'ов.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## Что такое priority classes?

**Priority Classes** - механизм, который позволяет назначать приоритеты POD'ам, чтобы Kubernetes мог управлять распределением ресурсов между POD'ами с разными приоритетами, особеннос в случае нехватки ресурсов.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class is for critical workloads"
```

## Что такое POD eviction?

**Eviction** - процесс удаления POD'ов с узлов, когда ресурсы на узле становятся дефицитными(нехватки памяти или диска).
Kubernetes может эвакуировать POD'ы с низким приоритетом, чтобы освободить ресурсы для более важных приложений.

```sh
kubectl drain <node-name> --ignore-daemonsets
```
