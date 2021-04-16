# Часть 0 | Подготовка инструментов и среды

**ОС:** windows 10

- Ставлю **kind**
    - Качаю exe, кладу в удобную папочку, создаю переменную среды

    ```bash
    curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.10.0/kind-windows-amd64
    ```

    - Запускаю команду `kind create cluster`

        ![homework-02-part0-screen01](https://user-images.githubusercontent.com/14264908/114895995-a041c280-9e18-11eb-9584-ddfb57ed3d35.png)

        Вроде все успешно

    - Похоже, что поторопился и надо было сначала почитать следующий слайд, после чего  указать файл конфиг и с него создавать. Удаляем кластер и снова создаем

        ```bash
        kind delete cluster
        kind create cluster --config=kind-config.yaml
        ```

        Не получается

        ```bash
        C:\_kuber\kind\otus>kind delete cluster
        Deleting cluster "kind" ...

        C:\_kuber\kind\otus>kind create cluster --config=kind-config.yaml
        ERROR: failed to create cluster: unknown apiVersion: kind.sigs.k8s.io/v1alpha3
        ```

        Что ж, поправим в нашем конфиге значение на то, которое в инструкции

        ```bash
        kind.**sigs.k8s**.io/v1alpha3 **->** kind.**x-k8s**.io/v1alpha3
        ```

        Не помогло

        ```bash
        C:\_kuber\kind\otus>kind create cluster --config=kind-config.yaml
        ERROR: failed to create cluster: unknown apiVersion: kind.x-k8s.io/v1alpha3
        ```

        Ладно, поставим и номер версии, как в инструкции

        ```bash
        kind.x-k8s.io/**v1alpha3 ->** kind.x-k8s.io/**v1alpha4**
        ```

        Заработало

        ```bash
        C:\_kuber\kind\otus>kind create cluster --config=kind-config.yaml
        Creating cluster "kind" ...
        • Ensuring node image (kindest/node:v1.20.2) �  ...
        ✓ Ensuring node image (kindest/node:v1.20.2) �
        • Preparing nodes �� �� �� �� �� ��   ...
        ✓ Preparing nodes �� �� �� �� �� ��
        • Configuring the external load balancer ⚖️  ...
        ✓ Configuring the external load balancer ⚖️
        • Writing configuration ��  ...
        ✓ Writing configuration ��
        • Starting control-plane �️  ...
        ✓ Starting control-plane �️
        • Installing CNI ��  ...
        ✓ Installing CNI ��
        • Installing StorageClass ��  ...
        ✓ Installing StorageClass ��
        • Joining more control-plane nodes ��  ...
        ✓ Joining more control-plane nodes ��
        • Joining worker nodes ��  ...
        ✓ Joining worker nodes ��
        Set kubectl context to "kind-kind"
        You can now use your cluster with:

        kubectl cluster-info --context kind-kind

        Thanks for using kind!
        ```

        ```bash
        C:\_kuber\kind\otus>kubectl get nodes
        NAME                  STATUS   ROLES                  AGE     VERSION
        kind-control-plane    Ready    control-plane,master   4m19s   v1.20.2
        kind-control-plane2   Ready    control-plane,master   3m34s   v1.20.2
        kind-control-plane3   Ready    control-plane,master   2m47s   v1.20.2
        kind-worker           Ready    <none>                 2m27s   v1.20.2
        kind-worker2          Ready    <none>                 2m27s   v1.20.2
        kind-worker3          Ready    <none>                 2m27s   v1.20.2
        ```
    - Но есть одно **но**. Опять, как и с миникубом видим ошибку

        ```bash
        > kubectl get cs 
        Warning: v1 ComponentStatus is deprecated in v1.19+
        NAME                 STATUS      MESSAGE                                                                                       ERROR
        scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
        controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
        etcd-0               Healthy     {"health":"true"}
        ```

        Надо бы поправить. Начнем править по очереди. Напомним команды

        ```bash
        sed -i '/- --port=0/d' /etc/kubernetes/manifests/kube-scheduler.yaml
        sed -i '/- --port=0/d' /etc/kubernetes/manifests/kube-controller-manager.yaml
        sudo systemctl restart kubelet.service
        ```

        Выполнил сначала на одном только `kind-control-plane` . Не помогло. Выполнил на остальных двух ему подобных. Помогло

        ```bash
        > kubectl get cs
        Warning: v1 ComponentStatus is deprecated in v1.19+
        NAME                 STATUS    MESSAGE             ERROR
        scheduler            Healthy   ok
        controller-manager   Healthy   ok
        etcd-0               Healthy   {"health":"true"} 
        ```

# Часть 1 | Пока не понятно

- **Воспроизвел** манифест
- **Применяю**. Получаю ошибку

    ```bash
    > kubectl apply -f frontend-replicaset.yaml
    error: error validating "frontend-replicaset.yaml": error validating data: ValidationError(ReplicaSet.spec): missing required field "selector" in io.k8s.api.apps.v1.ReplicaSetSpec; if you choose to ignore these errors, turn validation off with --validate=false
    ```

    Похоже отсутствует атрибут `selector` в спецификации объекта `ReplicaSet` . 

    Помнится, что `coredns` был связан с таким сетом. Глянем ка его описание. У него это поле выглядит так 

    ```bash
    # ...
    Selector:       k8s-app=kube-dns,pod-template-hash=74ff55c5b
    Labels:         k8s-app=kube-dns
                    pod-template-hash=74ff55c5b
    # ...
    ```

    Понятней не стало. Придется лезть в [документацию](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#pod-selector).

    Поле `.spec.selector` это указатель не лэйблы потенциально подходящего пода.

    В **ReplicaSet** `.spec.template.metadata.labels` должны соответствовать `.spec.selector`, иначе он будет отклонен API.

    Ну раз говорят, что они должны совпадать, то добавим, по примеру из документации

    ```yaml
    selector:
    	matchLabels:
    	  app: frontend
    ```

    Сработало

    ```bash
    > kubectl apply -f frontend-replicaset.yaml 
    replicaset.apps/frontend created
    ```

- **Смотрю** что получилось

    ```bash
    > get pod -l app=frontend
    NAME             READY   STATUS             RESTARTS   AGE
    frontend-wb2x4   0/1     CrashLoopBackOff   4          2m44s
    ```

    хммм... гляну как чего внутри пишут

    ```bash
    > kubectl logs frontend-wb2x4 
    {"message":"Tracing enabled.","severity":"info","timestamp":"2021-04-15T21:36:34.9170905Z"}
    {"message":"Profiling enabled.","severity":"info","timestamp":"2021-04-15T21:36:34.9173842Z"}
    {"message":"jaeger initialization disabled.","severity":"info","timestamp":"2021-04-15T21:36:34.9176803Z"}
    panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set

    goroutine 1 [running]:
    main.mustMapEnv(0xc0003ce000, 0xb1f32c, 0x1c)
            /src/main.go:259 +0x10b
    main.main()
            /src/main.go:117 +0x510
    ```

    ах, ну да, переменные же еще.

- **Исправил** переменные. Повторил. Теперь норм

    ```bash
    > kubectl get pod -l app=frontend
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-gtl4v   1/1     Running   0          12s
    ```

- **Увеличиваю** кол-во реплик сервиса

    ```bash
    > kubectl scale replicaset frontend --replicas=3
    replicaset.apps/frontend scaled

    > kubectl get rs frontend
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   3         3         3       3m53s
    ```

- **Проверяю** восстанавливаются ли поды

    ```bash
    > kubectl get pod -l app=frontend
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-g6n47   1/1     Running   0          2m11s
    frontend-gtl4v   1/1     Running   0          5m27s
    frontend-z922w   1/1     Running   0          2m11s

    > kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
    pod "frontend-g6n47" deleted
    pod "frontend-gtl4v" deleted
    pod "frontend-z922w" deleted

    > kubectl get pods -l app=frontend
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-d99wl   1/1     Running   0          17s
    frontend-mhb6j   1/1     Running   0          16s
    frontend-t729z   1/1     Running   0          16s

    > kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-d99wl   1/1     Running   0          3m50s
    frontend-mhb6j   1/1     Running   0          3m49s
    frontend-t729z   1/1     Running   0          3m49s
    frontend-d99wl   1/1     Terminating   0          3m50s
    frontend-mhb6j   1/1     Terminating   0          3m49s
    frontend-sspxz   0/1     Pending       0          0s
    frontend-sspxz   0/1     Pending       0          0s
    frontend-t729z   1/1     Terminating   0          3m49s
    frontend-qmg42   0/1     Pending       0          0s
    frontend-qmg42   0/1     Pending       0          0s
    frontend-xc5m2   0/1     Pending       0          0s
    frontend-qmg42   0/1     ContainerCreating   0          0s
    frontend-xc5m2   0/1     Pending             0          0s
    frontend-sspxz   0/1     ContainerCreating   0          0s
    frontend-xc5m2   0/1     ContainerCreating   0          0s
    frontend-d99wl   0/1     Terminating         0          3m50s
    frontend-mhb6j   0/1     Terminating         0          3m50s
    frontend-sspxz   1/1     Running             0          1s
    frontend-t729z   0/1     Terminating         0          3m50s
    frontend-xc5m2   1/1     Running             0          1s
    frontend-qmg42   1/1     Running             0          2s
    frontend-mhb6j   0/1     Terminating         0          3m55s
    frontend-mhb6j   0/1     Terminating         0          3m55s
    frontend-d99wl   0/1     Terminating         0          3m56s
    frontend-d99wl   0/1     Terminating         0          3m56s
    frontend-t729z   0/1     Terminating         0          3m55s
    frontend-t729z   0/1     Terminating         0          3m55s
    ```

- **Повторно применяю** манифест

    ```bash
    > kubectl apply -f frontend-replicaset.yaml
    replicaset.apps/frontend configured

    > kubectl get pod
    NAME             READY   STATUS        RESTARTS   AGE
    frontend-qmg42   0/1     Terminating   0          3m24s
    frontend-sspxz   0/1     Terminating   0          3m24s
    frontend-xc5m2   1/1     Running       0          3m24s

    > kubectl get replicaset            
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   1         1         1       13m 
    ```

- **Меняю** манифест для трех реплик: атрибут `replicas: 1` меняем на `replicas: 3`

    ```bash
    > kubectl apply -f frontend-replicaset.yaml
    replicaset.apps/frontend configured

    > kubectl get pod
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-9b8fg   1/1     Running   0          3s
    frontend-skprr   1/1     Running   0          3s
    frontend-xc5m2   1/1     Running   0          5m49s

    > kubectl get rs frontend
    NAME       DESIRED   CURRENT   READY   AGE
    frontend   3         3         3       15m
    ```

# Часть 2 | Обновляем образ в RS

- **Выпустил** новый тэг в хабе и поменял манифест на него
- **Применил**, подожал пару минут

    ```bash
    > kubectl apply -f frontend-replicaset.yaml | kubectl get pods -l app=frontend -w
    NAME             READY   STATUS    RESTARTS   AGE
    frontend-9b8fg   1/1     Running   0          12h
    frontend-skprr   1/1     Running   0          12h
    frontend-xc5m2   1/1     Running   0          12h
    ```

    Да, ничего не происходит

- Проверяю образ, указанный в RS:

    ```bash
    > kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
    'musisimaru/otus-hipstershop-01:0.0.2'

    > kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
    'musisimaru/otus-hipstershop-01:0.0.1 musisimaru/otus-hipstershop-01:0.0.1 musisimaru/otus-hipstershop-01:0.0.1'
    ```

    Опция `-o` - это output format
    Может применять значения
    `json` | `yaml` | `wide` | `name` | `custom-columns=...` | `custom-columns-file=...` | `go-template=...` | `go-template-file=...` | `jsonpath=...` | `jsonpath-file=...`

    [JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

    Очевидно образы отсались старыми

- Удаляем все созданные поды

    ```bash
    > kubectl delete pods -l app=frontend
    pod "frontend-9b8fg" deleted
    pod "frontend-skprr" deleted
    pod "frontend-xc5m2" deleted

    > kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
    'musisimaru/otus-hipstershop-01:0.0.2 musisimaru/otus-hipstershop-01:0.0.2 musisimaru/otus-hipstershop-01:0.0.2'
    ```

    Теперь образы новенькие

- **Вопрос:** Почему обновление **ReplicaSet** не повлекло обновление запущенных pod?

    **Ответ:** Потому что ***ReplicationController*** не проверяет соответствие запущенных Podов шаблону

# Часть 3 | Deployment

- **Собираю** образ и публикую для сервиса `paymentService` . Получили два тега `musisimaru/otus-hipstershop-02-paymentservice:0.0.1` и `musisimaru/otus-hipstershop-02-paymentservice:0.0.2`
- **Пишу** `paymentservice-replicaset.yaml` . Применяю. Разворачивается 3 пода.

    ```bash
    >kubectl get pod -l app=paymentservice -w
    NAME                   READY   STATUS              RESTARTS   AGE
    paymentservice-5cz9d   0/1     ContainerCreating   0          50s
    paymentservice-msctd   0/1     ContainerCreating   0          50s
    paymentservice-tx6j5   0/1     ContainerCreating   0          50s
    paymentservice-msctd   1/1     Running             0          57s
    paymentservice-5cz9d   1/1     Running             0          58s
    paymentservice-tx6j5   1/1     Running             0          59s
    ```

    **удаляю** rs

- **Копирую** `paymentservice-replicaset.yaml` переименовываю в `paymentservice-deployment.yaml` и меняю kind на `Deployment`. **Применяю**

    ```bash
    > kubectl apply -f paymentservice-deployment.yaml
    deployment.apps/paymentservice created 

    > kubectl get pod -l app=paymentservice -w
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-57b4459765-4n24z   1/1     Running   0          4s
    paymentservice-57b4459765-cvtqr   1/1     Running   0          4s
    paymentservice-57b4459765-hpgqb   1/1     Running   0          4s

    > kubectl get deployment 
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    paymentservice   3/3     3            3           18s

    > kubectl get rs -l app=paymentservice
    NAME                        DESIRED   CURRENT   READY   AGE
    paymentservice-57b4459765   3         3         3       2m14s
    ```

- Меняю версию образа. Применяю

    ```bash
    > kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-57b4459765-4n24z   1/1     Running   0          5m9s
    paymentservice-57b4459765-cvtqr   1/1     Running   0          5m9s
    paymentservice-57b4459765-hpgqb   1/1     Running   0          5m9s
    paymentservice-84895fbb4-gx9t4    0/1     Pending   0          0s
    paymentservice-84895fbb4-gx9t4    0/1     Pending   0          0s
    paymentservice-84895fbb4-gx9t4    0/1     ContainerCreating   0          0s
    paymentservice-84895fbb4-gx9t4    1/1     Running             0          3s
    paymentservice-57b4459765-cvtqr   1/1     Terminating         0          5m12s
    paymentservice-84895fbb4-lfkmv    0/1     Pending             0          0s
    paymentservice-84895fbb4-lfkmv    0/1     Pending             0          0s
    paymentservice-84895fbb4-lfkmv    0/1     ContainerCreating   0          0s
    paymentservice-84895fbb4-lfkmv    1/1     Running             0          2s
    paymentservice-57b4459765-hpgqb   1/1     Terminating         0          5m14s
    paymentservice-84895fbb4-8bxcb    0/1     Pending             0          0s
    paymentservice-84895fbb4-8bxcb    0/1     Pending             0          0s
    paymentservice-84895fbb4-8bxcb    0/1     ContainerCreating   0          0s
    paymentservice-84895fbb4-8bxcb    1/1     Running             0          3s
    paymentservice-57b4459765-4n24z   1/1     Terminating         0          5m17s
    paymentservice-57b4459765-cvtqr   0/1     Terminating         0          5m43s
    paymentservice-57b4459765-hpgqb   0/1     Terminating         0          5m45s
    paymentservice-57b4459765-4n24z   0/1     Terminating         0          5m48s
    paymentservice-57b4459765-cvtqr   0/1     Terminating         0          5m50s
    paymentservice-57b4459765-cvtqr   0/1     Terminating         0          5m50s
    paymentservice-57b4459765-hpgqb   0/1     Terminating         0          5m50s
    paymentservice-57b4459765-hpgqb   0/1     Terminating         0          5m50s
    paymentservice-57b4459765-4n24z   0/1     Terminating         0          6m 
    paymentservice-57b4459765-4n24z   0/1     Terminating         0          6m

    > kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'
    'musisimaru/otus-hipstershop-02-paymentservice:0.0.2 musisimaru/otus-hipstershop-02-paymentservice:0.0.2 musisimaru/otus-hipstershop-02-paymentservice:0.0.2'

    > kubectl get rs -l app=paymentservice                                                                                           
    NAME                        DESIRED   CURRENT   READY   AGE
    paymentservice-57b4459765   0         0         0       101m
    paymentservice-84895fbb4    3         3         3       96m

    > kubectl get rs -l app=paymentservice -o=jsonpath='{.items[0:2].spec.template.spec.containers[0].image}' 
    'musisimaru/otus-hipstershop-02-paymentservice:0.0.1 musisimaru/otus-hipstershop-02-paymentservice:0.0.2'
    ```

    Образ обновился. RS две, старая имеет старый образ и контролирует 0 подов 

- Смотрю историю версий Deployment

    ```bash
    > kubectl rollout history deployment paymentservice
    deployment.apps/paymentservice 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>
    ```

- Делаю откат

    ```bash
    > kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
    NAME                        DESIRED   CURRENT   READY   AGE
    paymentservice-57b4459765   0         0         0       107m
    paymentservice-84895fbb4    3         3         3       101m
    paymentservice-57b4459765   0         0         0       107m
    paymentservice-57b4459765   1         0         0       107m
    paymentservice-57b4459765   1         0         0       107m
    paymentservice-57b4459765   1         1         0       107m
    paymentservice-57b4459765   1         1         1       107m
    paymentservice-84895fbb4    2         3         3       101m
    paymentservice-57b4459765   2         1         1       107m
    paymentservice-84895fbb4    2         3         3       101m
    paymentservice-84895fbb4    2         2         2       101m
    paymentservice-57b4459765   2         1         1       107m
    paymentservice-57b4459765   2         2         1       107m
    paymentservice-57b4459765   2         2         2       107m
    paymentservice-84895fbb4    1         2         2       101m
    paymentservice-84895fbb4    1         2         2       101m
    paymentservice-57b4459765   3         2         2       107m
    paymentservice-57b4459765   3         2         2       107m
    paymentservice-84895fbb4    1         1         1       101m
    paymentservice-57b4459765   3         3         2       107m
    paymentservice-57b4459765   3         3         3       107m
    paymentservice-84895fbb4    0         1         1       101m
    paymentservice-84895fbb4    0         1         1       101m
    paymentservice-84895fbb4    0         0         0       101m

    > kubectl rollout history deployment paymentservice
    deployment.apps/paymentservice 
    REVISION  CHANGE-CAUSE
    2         <none>
    3         <none>

    > kubectl get rs -l app=paymentservice
    NAME                        DESIRED   CURRENT   READY   AGE
    paymentservice-57b4459765   3         3         3       111m
    paymentservice-84895fbb4    0         0         0       106m

    > kubectl get pods -l app=paymentservice
    NAME                              READY   STATUS    RESTARTS   AGE
    paymentservice-57b4459765-4ncpj   1/1     Running   0          5m18s
    paymentservice-57b4459765-b6qb9   1/1     Running   0          5m20s
    paymentservice-57b4459765-ngf6h   1/1     Running   0          5m19s
    ```

# Часть 4 ⭐ | Аналог blue-green и Reverse Rolling Update

Выдержки из документации

`maxSurge` - это необязательное поле определяющее максимальное кол-во подов, которое может быть создано сверх желаемого кол-ва подов. Значение может быть абсолютным кол-вом или в процентах желаемых подов  (например, 10%). Значение не может быть 0 если `MaxUnavailable` будет `0` . Кол-во вычисляется из процента путем округления в большую сторону. Значение по-умолчанию 25%.

Например, когда значение стоит 30% новый ReplicaSet может быть расширен немедленно когда начата rolling update, при этом абсолютное кол-во старых и новых подов не превышает 130% от желаемого кол-ва.

`Max Unavailable` - это необязательное поле определяющее максимальное кол-во подов, которые могут быть не доступны пока идет процесс обновления. Значение может быть абсолютным кол-вом или процентом от желаемых подов. Кол-во вычисляется из процента путем округления в меньшую сторону. Значение не может быть 0 если `maxSurge` будет `0` . Значение по-умолчанию 25%

Например, когда это значение задано 30%, то старый ReplicaSet может сузиться до 70% от желаемого кол-ва в тот же момент, как начинается rolling update. Как только новый под готов, старый ReplicaSet может сужаться дальше, преследуя расширение нового ReplicaSet, убеждаясь при этом, что полное кол-во достумных подов в любое время всего обновления не ниже 70% от желаемого кол-ва

## Аналог blue-green

По сути эта задача подразумевает, что не должно быть ни секунды когда наш сервис был бы не доступен. В здании приводится алгоритм - сначала развернулись все новые, потом свернулись все старые:

1. Развертывание трех новых pod
2. Удаление трех старых pod

Значит требуется в первую очередь указать `Max Unavailable` как `0` , что должно обеспечить ключевое условие подхода и указать `maxSurge` в значении 100%, чтобы условие удовлетворяло заданному в задании алгоритму. Пробуем.

Пробуем 

```bash
> kubectl apply -f paymentservice-deployment-bg.yaml | kubectl get pods -l app=paymentservice -w 
NAME                              READY   STATUS    RESTARTS   AGE
paymentservice-57b4459765-4ncpj   1/1     Running   0          75m
paymentservice-57b4459765-b6qb9   1/1     Running   0          76m
paymentservice-57b4459765-ngf6h   1/1     Running   0          75m
paymentservice-84895fbb4-mksjq    0/1     Pending   0          0s
paymentservice-84895fbb4-g7t2s    0/1     Pending   0          0s
paymentservice-84895fbb4-lww7n    0/1     Pending   0          0s
paymentservice-84895fbb4-mksjq    0/1     Pending   0          0s
paymentservice-84895fbb4-g7t2s    0/1     Pending   0          0s
paymentservice-84895fbb4-lww7n    0/1     Pending   0          0s
paymentservice-84895fbb4-g7t2s    0/1     ContainerCreating   0          1s
paymentservice-84895fbb4-lww7n    0/1     ContainerCreating   0          1s
paymentservice-84895fbb4-mksjq    0/1     ContainerCreating   0          1s
paymentservice-84895fbb4-mksjq    1/1     Running             0          2s
paymentservice-57b4459765-4ncpj   1/1     Terminating         0          76m
paymentservice-84895fbb4-g7t2s    1/1     Running             0          2s
paymentservice-84895fbb4-lww7n    1/1     Running             0          2s
paymentservice-57b4459765-b6qb9   1/1     Terminating         0          76m
paymentservice-57b4459765-ngf6h   1/1     Terminating         0          76m
paymentservice-57b4459765-4ncpj   0/1     Terminating         0          76m
paymentservice-57b4459765-ngf6h   0/1     Terminating         0          76m
paymentservice-57b4459765-b6qb9   0/1     Terminating         0          76m
paymentservice-57b4459765-4ncpj   0/1     Terminating         0          76m
paymentservice-57b4459765-4ncpj   0/1     Terminating         0          76m
paymentservice-57b4459765-ngf6h   0/1     Terminating         0          76m
paymentservice-57b4459765-ngf6h   0/1     Terminating         0          76m
paymentservice-57b4459765-b6qb9   0/1     Terminating         0          77m
paymentservice-57b4459765-b6qb9   0/1     Terminating         0          77m

> kubectl get pods -l app=paymentservice                                                     
NAME                             READY   STATUS    RESTARTS   AGE
paymentservice-84895fbb4-g7t2s   1/1     Running   0          3m5s
paymentservice-84895fbb4-lww7n   1/1     Running   0          3m5s
paymentservice-84895fbb4-mksjq   1/1     Running   0          3m5s
```

Вроде похоже

**Попробую** укзаать не в процентах а в абсолютном кол-ве. Все также сработало

## Reverse Rolling Update

Смысл видимо сделать так, чтобы каждый под менялся по очереди.

1. Удаление одного старого pod
2. Создание одного нового pod
3. ...

Также попробую и в числах и в процентах

**Выставлю** `maxSurge` в значение 0 - это обеспечит условие того, что сначала будем удалять, а потом создавать. А `Max Unavailable` укажем, как `1` или `35%`

**Пробую** с единицей

```bash
> kubectl apply -f paymentservice-deployment-reverse.yaml | kubectl get pods -l app=paymentservice -w
NAME                              READY   STATUS    RESTARTS   AGE
paymentservice-57b4459765-6bb8l   1/1     Running   0          3m44s
paymentservice-57b4459765-9lk42   1/1     Running   0          3m44s
paymentservice-57b4459765-fwd79   1/1     Running   0          3m44s
paymentservice-57b4459765-9lk42   1/1     Terminating   0          3m44s
paymentservice-84895fbb4-qfbhb    0/1     Pending       0          0s
paymentservice-84895fbb4-qfbhb    0/1     Pending       0          0s
paymentservice-84895fbb4-qfbhb    0/1     ContainerCreating   0          0s
paymentservice-84895fbb4-qfbhb    1/1     Running             0          2s
paymentservice-57b4459765-6bb8l   1/1     Terminating         0          3m46s
paymentservice-84895fbb4-hp8fs    0/1     Pending             0          0s
paymentservice-84895fbb4-hp8fs    0/1     Pending             0          0s
paymentservice-84895fbb4-hp8fs    0/1     ContainerCreating   0          0s
paymentservice-84895fbb4-hp8fs    1/1     Running             0          1s
paymentservice-57b4459765-fwd79   1/1     Terminating         0          3m47s
paymentservice-84895fbb4-j67hz    0/1     Pending             0          0s
paymentservice-84895fbb4-j67hz    0/1     Pending             0          0s
paymentservice-84895fbb4-j67hz    0/1     ContainerCreating   0          0s
paymentservice-84895fbb4-j67hz    1/1     Running             0          1s
paymentservice-57b4459765-9lk42   0/1     Terminating         0          4m15s
```

**Пробую** с 35%. Все также

# Часть 5 | Probes

**Написал** `т frontend-deployment.yaml` , добавил `readinessProbe` . Применил. Поднялись 3 пода.

**Смотрю** что написано. Есть вот такая вот строка

```yaml
Readiness: http-get http://:8080/_healthz delay=10s timeout=1s period=10s #success=1 #failure=3
```

**Меняю** в описании пробы `_healthz` на `_health` и тэг образа.

**Применяю**

```bash
> kubectl apply -f frontend-deployment.yaml | kubectl get pod -l app=frontend -w
NAME                        READY   STATUS    RESTARTS   AGE  
frontend-584bd48b44-b4g58   1/1     Running   0          5m35s
frontend-584bd48b44-lb57b   1/1     Running   0          5m35s
frontend-584bd48b44-rlprj   1/1     Running   0          5m35s
frontend-b5b97c669-w6fv7    0/1     Pending   0          0s
frontend-b5b97c669-w6fv7    0/1     Pending   0          0s
frontend-b5b97c669-w6fv7    0/1     ContainerCreating   0          0s
frontend-b5b97c669-w6fv7    0/1     Running             0          2s
```

Похоже завис на развертывании первого пода.

Вижу среди событий пода

```bash
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m13s               default-scheduler  Successfully assigned default/frontend-b5b97c669-w6fv7 to kind-worker2
  Normal   Pulled     2m12s               kubelet            Container image "musisimaru/otus-hipstershop-01:0.0.2" already present on machine
  Normal   Created    2m12s               kubelet            Created container server
  Normal   Started    2m12s               kubelet            Started container server
  Warning  Unhealthy  6s (x12 over 116s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
```

Смотрю статус

```bash
> kubectl rollout status deployment/frontend
Waiting for deployment "frontend" rollout to finish: 1 out of 3 new replicas have been updated...
```

# Часть 6 ⭐ | DaemonSet

**Взял** первый попавшийся `node-exporter-daemonset.yaml` с [гитхаба](https://github.com/prometheus-operator/kube-prometheus/blob/main/manifests/node-exporter-daemonset.yaml). 

**Создал** namespace `monitoring`

Применяю конфиг. Демон сет создался. Подов не видеть. Смотрю через `describe`. Вижу

```bash
Events:
  Type     Reason        Age                   From                  Message
  ----     ------        ----                  ----                  -------
  Warning  FailedCreate  48s (x16 over 3m32s)  daemonset-controller  Error creating: pods "node-exporter-" is forbidden: error looking up service account monitoring/node-exporter: serviceaccount "node-exporter" not found
```

Хмм... надо учетку создавать под него что ли =( Ладно, попробуем по аналогии как делал для дашборда.

```bash
> kubectl create serviceaccount node-exporter
serviceaccount/node-exporter created

> kubectl create clusterrolebinding node-exporter --clusterrole=cluster-admin --serviceaccount=default:node-exporter
clusterrolebinding.rbac.authorization.k8s.io/node-exporter created
```

**Удаляю** демонсет и пробую еще раз и получаю ту же самую ошибку, ведь я не в том namespace его завел.

**Заново**

```bash
> kubectl create serviceaccount node-exporter -n monitoring
serviceaccount/node-exporter created

> kubectl create clusterrolebinding node-exporter --clusterrole=cluster-admin --serviceaccount=monitoring:node-exporter
error: failed to create clusterrolebinding: clusterrolebindings.rbac.authorization.k8s.io "node-exporter" already exists
```

`clusterrolebinding` говорит такой есть. Возможно он не делится на namescpace. Проверим через `get` . Он конечно дает выполним команду и с пространством имен и без, но вывод одинаковый. Придется наверное дать другое имя

```bash
> kubectl create clusterrolebinding monitoring:node-exporter --clusterrole=cluster-admin --serviceaccount=monitoring:node-exporter
clusterrolebinding.rbac.authorization.k8s.io/monitoring:node-exporter created
```

Окей, вернемся к демонсету. А он тем, времинем починился, сразу, как я пользователя в правильном пространстве имен создал

```bash
Type     Reason            Age                     From                  Message
  ----     ------            ----                    ----                  -------
  Warning  FailedCreate      7m48s (x15 over 9m10s)  daemonset-controller  Error creating: pods "node-exporter-" is forbidden: error looking up service account monitoring/node-exporter: serviceaccount "node-exporter" not found
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-4xmbf
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-47rnr
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-7xmkk
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-t9xrb
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-9v8kr
  Normal   SuccessfulCreate  6m26s                   daemonset-controller  Created pod: node-exporter-xnlzq

>kubectl get pods -n monitoring 
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-47rnr   2/2     Running   0          7m38s
node-exporter-4xmbf   2/2     Running   0          7m38s
node-exporter-7xmkk   2/2     Running   0          7m38s
node-exporter-9v8kr   2/2     Running   0          7m38s
node-exporter-t9xrb   2/2     Running   0          7m38s
node-exporter-xnlzq   2/2     Running   0          7m38s
```

Прокину порты с выводом

```bash
> kubectl port-forward node-exporter-47rnr 9100:9100 -n monitoring
Forwarding from 127.0.0.1:9100 -> 9100
Forwarding from [::1]:9100 -> 9100
Handling connection for 9100
E0416 18:44:10.094800    6508 portforward.go:233] lost connection to pod
```

```bash
> curl localhost:9100/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary

# ... 100500 строк

# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 0
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```

Ну, вроде работает

# Часть 7 ⭐⭐ | DaemonSet

Добавил в `tolerations`. 

```yaml
- key: node-role.kubernetes.io/master
  effect: NoSchedule
```

Вот [тут](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#create-a-daemonset) пишут `this toleration is to have the daemonset runnable on master nodes`

Проверяю

```yaml
>kubectl apply -f node-exporter-daemonset.yaml | kubectl get pod -n monitoring -w
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-47rnr   2/2     Running   0          18m
node-exporter-4xmbf   2/2     Running   0          18m
node-exporter-7xmkk   2/2     Running   0          18m
node-exporter-9v8kr   2/2     Running   0          18m
node-exporter-t9xrb   2/2     Running   0          18m
node-exporter-xnlzq   2/2     Running   0          18m
node-exporter-9v8kr   2/2     Terminating   0          18m
node-exporter-9v8kr   0/2     Terminating   0          18m
node-exporter-9v8kr   0/2     Terminating   0          18m
node-exporter-9v8kr   0/2     Terminating   0          18m
node-exporter-gjlfb   0/2     Pending       0          0s
node-exporter-gjlfb   0/2     Pending       0          0s
node-exporter-gjlfb   0/2     ContainerCreating   0          0s
node-exporter-gjlfb   2/2     Running             0          2s
node-exporter-4xmbf   2/2     Terminating         0          18m
node-exporter-4xmbf   0/2     Terminating         0          18m
node-exporter-4xmbf   0/2     Terminating         0          18m
node-exporter-4xmbf   0/2     Terminating         0          18m
node-exporter-x47lx   0/2     Pending             0          0s
node-exporter-x47lx   0/2     Pending             0          0s
node-exporter-x47lx   0/2     ContainerCreating   0          0s
node-exporter-x47lx   2/2     Running             0          2s
node-exporter-47rnr   2/2     Terminating         0          18m
node-exporter-47rnr   0/2     Terminating         0          18m
node-exporter-47rnr   0/2     Terminating         0          18m
node-exporter-47rnr   0/2     Terminating         0          18m
node-exporter-vhksx   0/2     Pending             0          0s
node-exporter-vhksx   0/2     Pending             0          0s
node-exporter-vhksx   0/2     ContainerCreating   0          0s
node-exporter-vhksx   2/2     Running             0          2s
node-exporter-t9xrb   2/2     Terminating         0          18m
node-exporter-t9xrb   0/2     Terminating         0          18m
node-exporter-t9xrb   0/2     Terminating         0          19m
node-exporter-t9xrb   0/2     Terminating         0          19m
node-exporter-ht7qk   0/2     Pending             0          0s
node-exporter-ht7qk   0/2     Pending             0          0s
node-exporter-ht7qk   0/2     ContainerCreating   0          0s
node-exporter-ht7qk   2/2     Running             0          1s
node-exporter-xnlzq   2/2     Terminating         0          19m
node-exporter-xnlzq   0/2     Terminating         0          19m
node-exporter-xnlzq   0/2     Terminating         0          19m
node-exporter-xnlzq   0/2     Terminating         0          19m
node-exporter-v6n7h   0/2     Pending             0          0s
node-exporter-v6n7h   0/2     Pending             0          0s
node-exporter-v6n7h   0/2     ContainerCreating   0          0s
node-exporter-v6n7h   2/2     Running             0          2s
node-exporter-7xmkk   2/2     Terminating         0          19m
node-exporter-7xmkk   0/2     Terminating         0          19m
node-exporter-7xmkk   0/2     Terminating         0          19m
node-exporter-7xmkk   0/2     Terminating         0          19m
node-exporter-7hnzx   0/2     Pending             0          0s
node-exporter-7hnzx   0/2     Pending             0          0s
node-exporter-7hnzx   0/2     ContainerCreating   0          0s
node-exporter-7hnzx   2/2     Running             0          1s

> kubectl get pods -n monitoring -w 
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-7hnzx   2/2     Running   0          97s
node-exporter-gjlfb   2/2     Running   0          2m28s
node-exporter-ht7qk   2/2     Running   0          108s
node-exporter-v6n7h   2/2     Running   0          102s
node-exporter-vhksx   2/2     Running   0          119s
node-exporter-x47lx   2/2     Running   0          2m24s

> kubectl get pods -o=jsonpath='{.items[0:5].spec.nodeName}' -n monitoring
'kind-control-plane2 kind-worker kind-worker3 kind-control-plane3 kind-control-plane'
```