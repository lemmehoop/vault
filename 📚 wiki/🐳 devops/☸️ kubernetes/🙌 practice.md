### Pod
 - минимальная абстракция
 - может быть несколько контейнеров
	 - но нужно учитывать, что масштабироваться и развертываться они будут именно вместе!
	 - но например `redis` для кеша запустить вместе с приложением вполне нормально
 - нужно стараться держать один контейнер в поде
 - внутри всегда создается еще один контейнер в паузе (`k8s_POD`, который создает `net namespace`
	 - то есть например, у всех контейнеров будет общий `localhost`
##### QoS
- если на ноде мало места, то там скорее всего запустится только **Guaranteed**
- если на ноде не достает места, то первыми будут эвакуированы **BestEffort**
- самое защищаемое приложение - **Guaranteed**, остальные будет убивать первее
```bash
kubectl create -f pod.yml
kubectl describe pod <name>
kubectl get pods --show-labels

kubectl descrive pod <name> | grep QoS
# нет границ - BestEffort
# есть лимит и запрос, но не равны - Burstable
# равны - Guaranteed
```
***
```yml
---
apiVersion: v1   # версия api-server кубера
kind: Pod        # тип объекта кубера
metadata:
  name: podname  # имя пода
spec:            # техническое описание пода
  containers:
    - image: nginx:1.12
      name: nginx
      ports:
        - containerPort: 80  # указатель порта внутри сети кубера
```
### ReplicaSet
Абстракция над подами, чтобы было удобнее масштабироваться.
**labels** - метки, используются для фильтрации.
**selector** - определение, через которое ReplicaSet находит свои поды. Туда пишутся лейблы подов, которые сет будет считать своими.
Проблема его в том, что он не реагирует на обновление шаблона пода, а только поддерживает необходимое количество.
```bash
kubectl get replicaset
kubectl apply -f <config.yml>
kubectl scale replicaset myreplicaset --replicas=3
kubectl descibe replicaset <name>
```
***
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myreplicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx:1.12
          ports:
	        containerPort: 80

```
### Deployment
Это объект, который предоставляет декларативный способ обновления подов и их реплик. В файле описывается желаемое состояние, а затем при применении оно достигается.
`Pod` --> `ReplicaSet` --> `Deployment`
Процесс обновления:
- создает новый ReplicaSet, в котором будет все по нулям
- постепенно создаст в новом сете обновленные поды, а в старом - удалит
- старый ReplicaSet останется с нулями, чтобы при откате к нему вернуться
	- при `rollout undo` в старом произойдет обратный перенос подов
##### Probes
- **Liveness**
	- контроль за состоянием приложения,
	- исполняется всегда
- **Readiness**
	- проверка на готовность приложения принимать трафик
	- в случае неудачи приложением убирается из балансировки
	- проверяется всегда

```bash
kubectl get deployments
kubectl apply -f <file.yml>
kubectl set image deployment <name> '<containername/*=nginx:1.12'

kubectl rollout undo deployment <name> # откат к прошлой версии
kubectl rollout undo deployment <name> --to-revision=1  # на две версии назад

kubectl delete all --all
```
***
```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: my-deployment  
  labels:  
    app: my-deployment  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      app: my-app  
  strategy:  # способ обновления приложения  
    rollingUpdate:  # обновление подов по очереди, чтобы не было простоев  
      # значения тут можно задавать в процентах (X%), чтобы не обновлять по одному      maxSurge: 1  # сколько новых реплик можно создать при обновлении (например если была 1, то еще одну с новым образом)  
      maxUnavailable: 1  # какое максимальное количество реплик из заданного количества может быть недоступно  
    type: RollingUpdate  # есть еще Recreate - как Docker Compose  
  template:  
    metadata:  
      labels:  
        app: my-app  
    spec:  
      containers:  
        - name: nginx  
          image: nginx:1.13  
          imagePullPolicy: IfNotPresent  
          ports:  
            - containerPort: 80  
              protocol: TCP  
          readinessProbe:  
            failureThreshold: 3  # количество повторных запросов при ошибке  
            httpGet:  # путь и порт запроса  
              path: /  
              port: 80  
            periodSeconds: 10  # частота проверки  
            successThreshold: 1  # количество успешных проверок, чтобы обновить счетчики  
            timeoutSeconds: 1  # таймаут запроса  
          livenessProbe:  
            failureThreshold: 3  
            httpGet:  
              path: /  
              port: 80  
            periodSeconds: 10  
            successThreshold: 1  
            timeoutSeconds: 1  
            initialDelaySeconds: 10  # первую пробу спустя 10 секунд  
		  resources:  
		    requests:  
	          cpu: 50m  # 1/20 часть процессорного времени  
			  memory: 100Mi  
			limits:  
			  cpu: 100m  
			  memory: 120Mi
		    volumeMounts:  
		      - mountPath: /etc/nginx/conf.d  
		        name: config  
		volumes:  
		  - name: config  
		    configMap:  
		      name: my-configmap
```
### ConfigMap
Передача в приложение настроек.
Не нужно применять ничего, под раз в минуту сам смотрит и обновляет настройки. 
```bash
kubectl apply -f <first.yml> -f <second.yml>

kubectl port-forward <container name> <host port>:<pod port>
```
***
```yaml
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: my-configmap  
data:  
  default.conf: |  # обозначение многострочной переменной
      server {  
        listen 80 default_server;  
      
        default_type text/plain;  
      
        location / {  
          return 200;  
        }  
      }
```
***
```yaml
containers:
	volumeMounts:
		- name: config
		  mountPath: /etc/nginx/conf.d
volumes:
	- name: config
	  configMap:
		  name: my-configmap
```
### Secret
Как конфигмап, только информация хранится в закодированном виде. Для них можно определить права доступа.
- **generic** - пароли / токены для приложения
- **docker-registry**
- **tls** - сертификаты для **ingress**
```bash
kubectl create secret generic <name> --from-literal=<key>=<value>
```
### Service
Объект кластера, который позволяет сделать статичный выход от многих эфимерных подов в один доступный сервис, на существование которого можно положиться. То есть поды могут перезапускаться, удаляться, а сервис - нет.
При создании автоматически создается объект класса **Endpoints**, в котором прописаны IP-адреса подходящих подов. 
Адрес сервиса также может меняться, поэтому можно отправлять запросы по его имени.
```bash
kubectl get endpoints
```
***
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80  # host
      targetPort: 9376  # pod
```
### Ingress
Проксирование запросов снаружи сети кубера внуть в сервис и дальше в конкретные поды.
```bash
minikube addons enable ingress
minikube tunnel
```
***
```yaml
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
  name: my-ingress  
spec:  
  ingressClassName: nginx  
  rules:  
  - host: localhost  
    http:  
      paths:  
      - path: /  
        pathType: Prefix  
        backend:  
          service:  
            name: my-service  
            port:  
              number: 80
```
### PV/PVC
**PV (persistent volume)** - том (volume), на котором хранятся данные.
**PVC (persistent volume claim)** - запрос на подключение постоянного тома.
**Storage class** - конфигурация доступа к данным.
**PV Provisioner** - решает проблему, когда тома слишком большие, а требуется небольшое количество памяти (чтобы ради тома в 1 Гб не занимать 20 Гб). Он сам взаимодействует с **Storage Provider** и создает тома необходимого размера.
![[Pasted image 20250708211401.png]]
```bash
kubectl get pvc
kubectl get pv  # они живы, пока живы pvc. можно определить reclaim polciy, но обычно по умолчанию delete
```
***
```yaml
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: my-claim  
spec:  
  accessModes:  
    - ReadWriteOnce  # только один под может писать и читать, ReadWriteMany  
  resources:  
    requests:  
      storage: 1Gi
```
