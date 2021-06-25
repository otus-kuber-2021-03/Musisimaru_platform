# Часть 0 | Развернем среду

Ставим kind. 

Выполняем команду `export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"`

Получаем

```bash
$ export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
ERROR: unknown flag: --name

$ export KUBECONFIG="$(kind get kubeconfig-path --name='kind')"
ERROR: unknown flag: --name
```

# Часть 1 | StatefulSet c MinIO

Применили манифест по указанной ссылке *(в задании ссылка не верная)*

```bash
> kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-87d81364-eca8-4c7f-8945-7af0c96f78c5   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                113s

> kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-87d81364-eca8-4c7f-8945-7af0c96f78c5   10Gi       RWO            standard       2m18s

> kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          8m4s

> kubectl get statefulsets
NAME    READY   AGE
minio   1/1     8m32s
```

Поставил клиент mc.exe. Вроде все работает.

# Часть * | Спрятать данные в секреты

Взял значения для переменных среды `MINIO_ACCESS_KEY` и `MINIO_SECRET_KEY`, перевел в base64 и сохранил в один секрет под разными ключами. Манифест приложил в папку д.з.