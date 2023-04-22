## deployment
### 1. Создадим deployment
Сначала проверим, что ни одного deployment'а у нас не создано, для этого выполним команду:
```
kubectl get deployments
```
Получим вывод, который скажет, что в нашем кластере на данный момент deployment'ов нет:
```
No resources found in default namespace.
```
Создадим первый наш deployment при помощи командной строки, опять с nginx и открытым портом 80:
```
kubectl create deployment nginx-deployment --image nginx:1.14.2 --port=80
```
Kubectl ответит, что deployment создан:
```
deployment.apps/nginx-deployment created
```
Так как deployment создался, давайте посмотрим как выглядит вывод команды ```kubectl get deployments```:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           42s
```
Вместе с deployment'том создался pod, его статус можно проверить уже знакомой командой ```kubectl get pods```:
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-8549b88f96-z25th   1/1     Running   0          2m2s
```
А так же создался новый объект replicaset, в информации которого можно узнать количество реплик у pod'а, сколько реплик работает на текущий момент и другую информацию. \
Команда чтобы посмотреть его ```get rs```:
```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-8549b88f96   1         1         1       3m1s
```
Из вывода стало понятно что у нас работает всего 1 pod, реплик нет.
### 2. Сделаем scale
Scale'инг даст нам HA. Для создания трех реплик нашего deployment'та выполним команду:
```
kubectl scale deployment nginx-deployment --replicas 3
```
Получим вывод:
```
deployment.apps/nginx-deployment scaled
```
Давайте посмотрим наши pod'ы, после вывода команды ```kubectl get pods```, мы увидим что их стало три:
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-8549b88f96-6bvc8   1/1     Running   0          5s
nginx-deployment-8549b88f96-x8hnj   1/1     Running   0          5s
nginx-deployment-8549b88f96-z25th   1/1     Running   0          10m
```
Теперь посмотрим как изменился наш объект-deployment, введем команду ```kubectl get deployments```, вывод будет примерно таким:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           14m
```
Мы видим, что у нас работает 3 из 3 pod'ов, все доступны и работают уже 14 минут.
### 3. Удалим один pod
Теперь удалим последний из pod'ов в реплике и посмотрим что произойдет. В примере это pod с именем ```nginx-deployment-8549b88f96-z25th```, у вас же будет другое имя.
```
kubectl delete pods nginx-deployment-8549b88f96-z25th
```
Вывод будет таким:
```
pod "nginx-deployment-8549b88f96-z25th" deleted
```
Давайте проверим сколько сейчас у нас осталось pod'ов с приложением nginx, ```kubectl get pods```:
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-8549b88f96-6bvc8   1/1     Running   0          20m
nginx-deployment-8549b88f96-8fzts   1/1     Running   0          7s
nginx-deployment-8549b88f96-x8hnj   1/1     Running   0          20m
```
Вывод показывает, что у нас количество pod'ов не изменилось. Вместо удаленного pod'а, k8s практически сразу создал новый с именем ```nginx-deployment-8549b88f96-8fzts```.
### 4. Обновим контейнер в deployment'е
Внезапно наша служба ИБ разрешила использовать в периметре версию nginx 1.17.2, нам требуется обновить уже развернутый контейнер в deployment'е на новый. \
Для начала Вам необходимо вывести информацию о обновляемом deployment'е, чтобы узнать название обновляемого контейнера. Сделайте это сами. \
Теперь давайте посмотрим history нашего deployment'a, наберите команду:
```
kubectl rollout history deployment/nginx-deployment
```
В выводе мы сможем посмотреть версию и изменения в ней. Сейчас мы видим версию (ревизию) 1 без изменений <none>:
```
REVISION  CHANGE-CAUSE
1         <none>
```
Начнем обновление образа, введите команду ниже, где <IMAGE_NAME> - имя нашего контейнера который мы обновляем:
```
kubectl set image deployment/nginx-deployment IMAGE_NAME=nginx:1.15.6
``` 
Вывод:
```
deployment.apps/nginx-deployment image updated
```
K8s начнет деплоить новую версию контейнера, сам процесс можете посмотреть командой:
``` 
kubectl rollout status deployment/nginx-deployment  
```
Вывод будет примерно таким:
``` 
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```
Обязательно запишим лог в history:
```
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="version change to 1.14.2 to 1.15.6" --overwrite=true
```
И тут мы понимаем что развернули не ту версию приложения! Надо было 1.17.2, а мы задеплоили 1.15.6! \
Давайте сейчас развернем правильную версию:
```
kubectl set image deployment/nginx-deployment IMAGE_NAME=nginx:1.17.2
```
Запишим в history:
```
kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="version change to 1.15.6 to 1.17.2" --overwrite=true
```
Давайте выведем текущее состояние history:
```
kubectl rollout history deployment/nginx-deployment
```
Вывод будет примерно таким:
``` 
REVISION  CHANGE-CAUSE
1         <none>
2         version change to 1.14.2 to 1.15.6
3         version change to 1.15.6 to 1.17.2
```
Проверим, а обновилось ли наше приложение на версию 1.17.2 в действителности:
```
kubectl describe deployment nginx-deployment
```
Вывод будет действительно большим, Вам нужно найти следующую информацию, убедиться что версия приложения 1.17.2.
```
...  
 Containers:
   nginx:
    Image:        nginx:1.17.2
    Port:         80/TCP 
...
```
Внезапно к нам прибегает специалист ИБ и говорит что он ошибся, версия приложения должна быть 1.15.6! А мы то помним, что деплоили ее ранее, и в history deployment она была 2 ревизией, давайте к ней и вернемся! (У Вас версия ревизии может отличаться)
```
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```
Давайте проверим, что получилось. Выполним знакомую нам команду ```kubectl describe deployment nginx-deployment```:
```
...  
 Containers:
   nginx:
    Image:        nginx:1.15.6
    Port:         80/TCP 
...
```
### 5.Создадим deployment из манифест-файла
Для начала давайте удалим все ранее созданные deployment'ы:
```
kubectl delete deployments --all
```
Можете проверить командой get, созданные ранее объекты на все deployment'ы, такие как: pod'ы, replicaset и сам deployment, будут удалены.
Команда для создания deployment'а при помощи манифеста приведена ниже:
```
kubectl apply -f https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/4.%20Deployment/nginx-deployment.yaml
```
Для просмотра содержимого yaml-файла просто перейдите по ссылке "https://raw.githubusercontent.com/Ramazeca/sirius-k8s-lessons/main/4.%20Deployment/nginx-deployment.yaml". \
Давайте посмотрим на вновь созданные объекты, для начала это будет сам deployment: ```kubectl get deployment```:
```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment-yaml   3/3     3            3           17s 
```
Далее посмотрим на replicaset: ```kubectl get rs```: 
```
NAME                               DESIRED   CURRENT   READY   AGE
nginx-deployment-yaml-85996f8dbd   3         3         3       4m
```
Проверим pod'ы: ```kubectl get pods```:
```
NAME                                     READY   STATUS    RESTARTS   AGE
nginx-deployment-yaml-85996f8dbd-n4lm2   1/1     Running   0          12m
nginx-deployment-yaml-85996f8dbd-pdxb6   1/1     Running   0          12m
nginx-deployment-yaml-85996f8dbd-wf86g   1/1     Running   0          12m
```  
### Дополнительные команды
```
kubectl autoscale deployment nginx-deployment --min=3 --max=5 --cpu-percent=70 := пример создания объекта HorizontalPodAutoscaler (hpa);
kubectl describe deployment DEPLOYMENT_NAME := вывести информацию о deployment'е;
kubectl rollout undo deployment/nginx-deployment := возврат на предыдущую версию приложения;
```
