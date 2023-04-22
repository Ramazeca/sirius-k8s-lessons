## services

Нам необходим будет deployment, мы будем использовать nginx-deployment-yaml из предыдущего урока. \
Вам может понадобиться ssh-клиент putty, winscp или другой.

### 1. ClusterIP
ClusterIP открывает service **на внутреннем** IP-адресе кластера k8s. При выборе этого значения, service становится доступен только изнутри кластера. ClusterIP - service, который используется по умолчанию, если явно не указывать тип для service. Нагрузка балансируется между всеми worker node'ами.
```
kubectl expose deployment nginx-deployment-yaml --type=ClusterIP --port 80
```
Вывод будет таким:
```
service/nginx-deployment-yaml exposed
```
Вместе с service, k8s создался еще один объект под названием endpoints, который говорит service'су на какие pod'ы необходимо направлять трафик. Сначала давайте посмотрим на объект endpoints:
```
kubectl get endpoints //можно использовать alias ep
```
Вывод будет примерно таким:
```
NAME                    ENDPOINTS                                      AGE
kubernetes              172.24.104.56:8443                             24m2s
nginx-deployment-yaml   10.244.0.34:80,10.244.0.35:80,10.244.0.36:80   15s
```
Из вывода видно, что создался объект endpoint с именем как у нашего deployment'а, у которого в ENDPOINTS прописаны все IP pod'ов с открытым портом 80, на всех pod'ах. \
Вы можете проверить IP pod'ов при помощи команды ```kubectl describe pods``` или ```kubectl get pods -o wide```. \
Теперь проверим статус созданного сервиса:
```
kubectl get services
```
Вывод:
```
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP   25m
nginx-deployment-yaml   ClusterIP   10.108.196.81   <none>        80/TCP    40s
```
K8s говорит что теперь по IP 10.108.196.81:80 мы можем достучаться до нашего приложения с любой worker node кластера k8s, давайте проверим. \
Так как в состав minikube входит control plane и worker node, давайте зайдем на него по ssh используя login/password: docker/tcuser. IP кластера можно узнать командой:
```
minikube ip
```
или командой ```minikube ssh```. \
Успешная авторизация в minikube выглядит так: \
 \
![pic_welcome_to_minikube](https://github.com/Ramazeca/sirius-k8s-lessons/blob/7ce46f3108fc4d84e27fc36e9c91a23e730cc63d/5.%20Services/pic_welcome_to_minikube.png "Welcome to minikube") \
 \
Введите команду client url:
```
curl 10.108.196.81
```
Если сделано все правильно, мы должны увидеть ответ в html формате, как ниже: 
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Удалите service типа ClusterIP.
### 2. NodePort
NodePort предоставляет service **на IP-адресе каждого узла (worker node) через статический порт**. Чтобы сделать порт узла доступным, k8s устанавливает IP-адрес кластера, такой же, как если бы вы запросили службу типа: ClusterIP.
```
kubectl expose deployment nginx-deployment-yaml --type=NodePort --port 80
```
Проверим статус созданного сервиса:
```
kubectl get services //можно использовать alias svc
```
Вывод:
```
NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP   10.96.0.1     <none>        443/TCP        30m
nginx-deployment-yaml   NodePort    10.98.103.1   <none>        80:32075/TCP   4s
```
Из вывода понятно, k8s создал service типа NodePort с открытым внешним портом 32075 (у вас будет другой порт) на каждом worker node, а так же создал CLUSTER-IP с IP 10.98.103.1. \
Давайте проверим. Мы помним что minikube это одноузловой кластер k8s и IP адрес его мы уже узнавали командой ```minikube ip```. \
Теперь откройте браузер и введите:
```
ВАШ_IP_АДРЕС_MINIKUBE:ВАШ_ПОРТ
```
Если все правильно сделали, откроется страница "Welcome to nginx!". \
Удалите service типа NodePort.
### 3. ExternalName
Данный тип service не проксируется в кластер, поэтому не имеет ClusterIP и позволяет авторам приложений ссылаться на службы, существующие вне платформы, на других кластерах. 
Service'ы ExternalName реализуются исключительно на уровне DNS – для службы создается простая DNS-запись CNAME. Поэтому клиенты, подключающиеся к service'у, будут подключаться к внешнему service'у напрямую, полностью минуя службный прокси.
```
kubectl create service externalname nginx-deployment-yaml --external-name api.nginx-deployment-yaml.com
```
Теперь если сделать ```kubectl get services```, мы увидим FQDN в поле EXTERNAL-IP, на которое будет ссылаться наше приложение.
```
NAME                    TYPE           CLUSTER-IP   EXTERNAL-IP                     PORT(S)   AGE
kubernetes              ClusterIP      10.96.0.1    <none>                          443/TCP   35m
nginx-deployment-yaml   ExternalName   <none>       api.nginx-deployment-yaml.com   <none>    4s
```
Удалите service типа ExternalName.
### 4. LoadBalancer
Service LoadBalancer является "расширением" service'а ClusterIP и работает только в облачных кластерах k8s. По сути в облаке создается балансировщик нагрузки, который будет перенаправлять все подключения в service NodePort. Следовательно, вы сможете получить доступ к service'у через IP-адрес балансировщика нагрузки.
```
kubectl expose deployment nginx-deployment-yaml --type=LoadBalancer --port 80
```
Посмотрим что получилось через ```kubectl get services```:
```
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP      10.96.0.1      <none>        443/TCP        13d
nginx-deployment-yaml   LoadBalancer   10.96.29.126   <pending>     80:32560/TCP   10s
```
Мы видим, создался service типа LoadBalancer со статусом <pending>. Service ожидает поднятия балансировщика нагрузки. Если это проделывать в k8s облаке, создастся балансировщик нагрузки с внешним IP адресом, который будет перенаправлять трафик на pod'ы deployment'а nginx-deployment-yaml. \
Удалите service типа LoadBalancer.
### Дополнительные команды
```
ssh LOGIN@IP := инициация подключения к серверу по ssh в Linux;
kubectl delete services SERVICE_NAME := удаление сервиса k8s;
```
