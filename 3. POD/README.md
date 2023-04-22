## pod
### 1. Создадим pod
Для начала проверим, что ни одного pod у нас не создано, для этого выполним команду:
```
kubectl get pods
```
Получим вывод, который скажет, что в нашем кластере на данный момент pod'ов нет:
```
No resources found in default namespace.
```
Теперь создадим первый наш pod при помощи командной строки, на борту будет nginx, у которого будет открыт порт 80, так как это web-сервер:
```
kubectl run nginx --image=nginx:1.14.2 --port=80
```
Получим вывод, который говорит нам, что pod nginx создан:
```
pod/nginx created
```
Теперь если мы введем команду:
```
kubectl get pods
```
Получим вывод в табличной форме, который покажет нам имя нашего pod'а, количество контейнеров, статус и другие параметры:
```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          18s
```
### 2. Зайдем на pod
Чтобы войти в shell pod'а, введем команду:
```
kubectl exec -it nginx -- sh
```
Что бы выйти из pod'а наберите в терминале ```exit```.
### 3. Удалим pod
Комманда удаления pod'а выгдядит так:
```
kubectl delete pods nginx
```
Теперь проверим, что на кластере нет нашего pod, опять введем команду ```kubectl get pods```, вывод должен быть таким:
```
No resources found in default namespace.
```
### 4. Создадим pod из манифест-файла
Команда для создания pod'а при помощи манифеста приведена ниже:
```
kubectl apply -f https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/3.%20POD/simple-pod.yaml
```
Для просмотра содержимого yaml-файла просто перейдите по ссылке "https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/3.%20POD/simple-pod.yaml". \
Теперь проверим создался ли наш pod, команда уже встречалась ранее:
```
kubectl get pods
```
Вывод будет примерно таким:
```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          10s
```
### Дополнительные команды
```
kubectl describe pods POD_NAME := посмотреть дополнительные сведения по pod'у;
kubectl exec -it POD_NAME -- sh := войти в pod и запустить внутри шелл;
kubectl port-forward POD_NAME 1234:80 := переадресовать порт 1234 в локальной машине на порт 80 в pod'е;
kubectl logs POD_NAME := вывод лога pod'а.
```
