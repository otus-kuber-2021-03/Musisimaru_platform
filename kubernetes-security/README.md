# Часть 0 | Развернем среду

Разворачивать буду **Kind**, как и в прошлой домашке. Как-то он стабильнее себя вел.

Утсановил кластер с настройками по-умолчанию, уже традиционно поправил `kube-scheduler.yaml` и `kube-controller-manager`. Готово

```bash
> kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

Попробуем может еще раз развернуть k9s. Может в этот раз заведется.

# Часть 1 | Task 01

## Формулировка задачи

- Создать `Service Account` **bob**, дать ему роль **admin** в рамках всего кластера
- Создать `Service Account` **dave** без доступа к кластеру

...

# Часть 2 | Task 02

## Формулировка задачи

- Создать `Namespace` **prometheus**
- Создать `Service Account` **carol** в этом `Namespace`
- Дать всем `Service Account` в `Namespace` **prometheus** возможность делать `get`, `list`, `watch` в отношении `Pods` всего кластера

...

# Часть 3 | Task 03

- Создать `Namespace` **dev**
- Создать `Service Account` **jane** в `Namespace` **dev**
- Дать `jane` роль **admin** в рамках `Namespace` **dev**
- Создать `Service Account` **ken** в `Namespace` **dev**
- Дать **ken** роль **view** в рамках `Namespace` **dev**

...