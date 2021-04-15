# Часть 0 | Подготовка инструментов и среды

**ОС:** windows 10

- Ставлю **kind**
    - Качаю exe, кладу в удобную папочку, создаю переменную среды

    ```bash
    curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.10.0/kind-windows-amd64
    ```

    - Запускаю команду `kind create cluster`

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2313e3a4-a1c8-4660-8997-26f3c009735f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2313e3a4-a1c8-4660-8997-26f3c009735f/Untitled.png)

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

    -