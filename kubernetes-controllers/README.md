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

-
    -
