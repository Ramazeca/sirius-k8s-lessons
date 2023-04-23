## minikube
### 1. Установка minikube
Перейдите на официальную страницу K8S и выберите шаги установки которые подходят для Вашей операционной системы:
```
https://kubernetes.io/ru/docs/tasks/tools/install-minikube/
```
### 2. Запуск и проверка работы minikube
Запустите minikube, одноузловой кластер K8S создастся автоматически:
```
minikube start [--cpus=<int> --memory=<int>gb/mb --disk-size=<>gb/mb]
```
Вывод будет примерно следующим:
```
* Starting control plane node minikube in cluster minikube
* Creating hyperv VM (CPUs=2, Memory=6000MB, Disk=20000MB) ...
* Preparing Kubernetes v1.26.1 on Docker 20.10.23 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring bridge CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
Теперь проверим статус minikube:
```
minikube status
```
Вывод показан ниже:
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
Командой ниже посмотрим список и статус всех служебных Pod'ов одноузлового кластера minikube:
```
kubectl get pods --all-namespaces
```
Вывод:
```
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-787d4945fb-cbkvg           1/1     Running   0             10m11s
kube-system   etcd-minikube                      1/1     Running   0             10m11s
kube-system   kube-apiserver-minikube            1/1     Running   0             10m11s
kube-system   kube-controller-manager-minikube   1/1     Running   0             10m11s
kube-system   kube-proxy-5clh9                   1/1     Running   0             10m11s
kube-system   kube-scheduler-minikube            1/1     Running   0             10m11s
kube-system   storage-provisioner                1/1     Running   0             10m11s
```
### Дополнительные команды
```
minikube addons list := список дополнений в minikube;
minikube delete := удалить minikube (так же удаляется ВМ);
minikube dashboard := открыть дашборд;
minikube ssh := войти на ВМ minikube, пользователь - docker;
kubectl version --short := вывод версий компонентов кластера k8s;
kubectl cluster-info := вывод статуса компонентов кластера k8s;
kubectl get --raw='/readyz?verbose' := более подробный вывод статуса компонентов кластера;
kubectl get nodes := вывод хостов кластера k8s;
kubectl describe node MY_NODE := посмотреть дополнительные сведения по хосту кластера;
kubectl api-resources := посмотреть информацию по объектам API k8s.
```
Если все получилось, переходим к знакомству с Pod'ами.
