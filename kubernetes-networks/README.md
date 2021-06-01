# Часть 0 | Подготовка инструментов и среды

По заданию ставим `minikube`. В этот раз попробуем через Hyper V.

Для начала создадим виртуальный свитч

```bash
> choco install minikube                                                                                                                Chocolatey v0.10.15                                                                                                                                         Installing the following packages:
minikube
By installing you accept licenses for the packages.
Progress: Downloading Minikube 1.20.0... 100%

Minikube v1.20.0 [Approved]
minikube package files install completed. Performing other installation steps.
 ShimGen has successfully created a shim for minikube.exe
 The install of minikube was successful.
  Software install location not explicitly set, could be in package or
  default install location if installer.

Chocolatey installed 1/1 packages.
 See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).

> minikube start --vm-driver hyperv --hyperv-virtual-switch "Primary Virtual Switch"
😄  minikube v1.20.0 on Microsoft Windows 10 Pro For Workstations 10.0.19042 Build 19042
✨  Using the hyperv driver based on user configuration
💿  Downloading VM boot image ...
    > minikube-v1.20.0.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
    > minikube-v1.20.0.iso: 245.40 MiB / 245.40 MiB [ 100.00% 11.02 MiB p/s 22s
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.20.2 preload ...
    > preloaded-images-k8s-v10-v1...: 491.71 MiB / 491.71 MiB  100.00% 11.10 Mi
🔥  Creating hyperv VM (CPUs=2, Memory=6000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.6 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Проверяем работу

```bash
> minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

> kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```

Кластер отзывается, проблема все та же. Открываем виртуалку. У нас запрашивают учетные данные.

username: “docker“, password: “tcuser“

Исправляем конфиги

```bash
sudo sed -i '/- --port=0/d' /etc/kubernetes/manifests/kube-scheduler.yaml
sudo sed -i '/- --port=0/d' /etc/kubernetes/manifests/kube-controller-manager.yaml
sudo systemctl restart kubelet.service
```

Проверяем

```bash
> kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

Все работает

## Материалы

[Minikube on Windows 10 with Hyper-V](https://medium.com/@JockDaRock/minikube-on-windows-10-with-hyper-v-6ef0f4dc158c)

# Часть 1 | ...