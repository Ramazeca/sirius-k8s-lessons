## ingress

ingress - объект API k8s, управляющий внешним доступом к service'ам в кластере, обычно HTTP/HTTPS. \
Маршрутизация трафика контролируется правилами, определенными на ресурсе Ingress.

В рамках стрима будем использовать [NGINX Ingress controller](https://docs.nginx.com/nginx-ingress-controller).

### 1. Включение ingress controller'а
Для включения NGINX Ingress controller на Minikube выполните следующую команду:
```
minikube addons enable ingress
```
При успешном включении ingress controller'а вывод будет примерно таким:
```
* ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
  - Using image registry.k8s.io/ingress-nginx/controller:v1.5.1
* Verifying ingress addon...
* The 'ingress' addon is enabled
```
Убедимся, что NGINX Ingress controller запущен
```
kubectl get pods -n ingress-nginx
```
Вывод будет примерно таким:
```
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-hc4db       0/1     Completed   0          3m
ingress-nginx-admission-patch-79dkr        0/1     Completed   0          3m
ingress-nginx-controller-77669ff58-qlfnx   1/1     Running     0          3m
```
### 2. Создание deployment'а 1
Создадим deployment web с помощью следующей команды:
```
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```
Вывод должен быть следующим:
```
deployment.apps/web created
```
Поднимем service типа NodePort, порт 8080:
```
kubectl expose deployment web --type=NodePort --port=8080
```
Вывод будет таким:
```
service/web exposed
```
Убедимся, что service создан и доступен на порте узла:
```
kubectl get service web
```
Вывод будет примерно таким:
```
NAME   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.96.147.115   <none>        8080:32405/TCP   1m13s
```
Как видим тип service'а NodePort, значит на всех node'ах открылся какой-то порт, в нашем случает это 32405. Давайте попробуем зайти на него, командой узнаем url:
```
minikube service web --url
```
Получим вывод и попробуем достучаться до приложение через браузер (у Вас будет другие IP/port):
```
http://172.24.104.56:32405
```
Вывод в браузере будет примерно такой:
```
Hello, world!
Version: 1.0.0
Hostname: web-68487bc957-8zjdd
```
Мы получили доступ к образцу приложения через IP-адрес Minikube и NodePort. Далее мыполучим доступ к приложению с помощью ресурса Ingress.
### 3. Создание ingress'а
Создим объект Ingress, применив манифест из файла [my-ingress.yaml](https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/6.%20Ingress/my-ingress.yaml):
```
kubectl apply -f https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/6.%20Ingress/my-ingress.yaml
```
Вывод будет таким:
```
ingress.networking.k8s.io/my-ingress created
```
Убедимся что IP установлен:
```
kubectl get ingress //можно использовать alias ing
```
Вы должны увидеть адрес IPv4 в столбце ADDRESS, например:
```
NAME         CLASS   HOSTS              ADDRESS         PORTS   AGE
my-ingress   nginx   hello-world.info   172.24.104.56   80      1m54s
```
Давайте проверим что ingress controller направляет трафик:
```
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
```
Вывод будет примерно таким:
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    60  100    60    0     0  12099      0 --:--:-- --:--:-- --:--:-- 15000HTTP/1.1 200 OK
Date: Sat, 01 Apr 2023 16:10:43 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 60
Connection: keep-alive

Hello, world!
Version: 1.0.0
Hostname: web-68487bc957-cbx7w
```
Так же Вы можете открыть браузер и перейти прямой ссылкой http://hello-world.info, но перед этим не забудьте добавить DNS-запись в файл hosts. \
Расположение файла hosts в linux, windows:
```
linux: /etc/hosts
windows: <буква системного диска>:\Windows\System32\drivers\etc
```
### 4. Создание deployment'а 2
Давайте создадим еще один deployment, выполните команду:
```
kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
```
Вывод будет таким:
```
deployment.apps/web2 created
```
Создадим второй объект service типа NodePort:
```
kubectl expose deployment web2 --port=8080 --type=NodePort
```
Вывод будет таким:
```
service/web2 exposed
```
Давайте проверим, у нас должно быть развернуто два deplyment'а:
```
kubectl get deployments //можно использовать alias deploy
```
Если все правильно, вывод должен быть примерно таким:
```
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           10m
web2   1/1     1            1           1m19s
```
### 5. Создание ingress'а 2
Создадим второй объект ingress, отредактируйте существующий манифест my-ingress.yaml и добавьте следующие строки в конец:
```
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: web2
                port:
                  number: 8080
```
При редактировании файла будьте внимательны, придерживайтесь синтаксиса yaml и примените изменение.
```
kubectl apply -f https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/6.%20Ingress/my-ingress.yaml
```
Вывод будет таким:
```
ingress.networking.k8s.io/my-ingress configured
```
### 6. Проверка ingress'а
Проверим доступ к версии 1.0.0 приложения:
```
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
```
Вывод будет примерно такой:
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    60  100    60    0     0   2635      0 --:--:-- --:--:-- --:--:--  2727HTTP/1.1 200 OK
Date: Sat, 01 Apr 2023 17:28:44 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 60
Connection: keep-alive

Hello, world!
Version: 1.0.0
Hostname: web-68487bc957-cbx7w
```
Из вывода видно, что наше приложение версии 1.0.0 доступно снаружи и оно распологается на pod'е web-68487bc957-cbx7w. \
Теперь проверим доступ к нашему приложению версии 2.0.0 выполнив команду:
```
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info/v2
```
Вывод:
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    61  100    61    0     0  11505      0 --:--:-- --:--:-- --:--:-- 12200HTTP/1.1 200 OK
Date: Sat, 01 Apr 2023 17:36:39 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 61
Connection: keep-alive

Hello, world!
Version: 2.0.0
Hostname: web2-6459878f46-jbdmk
```
Приложение версии 2.0.0 тоже доступно и мы подключились к pod'у web2-6459878f46-jbdmk. \
Так же для проверки доступности можно использовать браузер, если Вы добавили записи в файл hosts. Откройте Ваш браузер и зайдите по адресам соответствующим версиям приложений: \
Версия 1.0.0 -> http://hello-world.info \
Версия 2.0.0 -> http://hello-world.info/v2


