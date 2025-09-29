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
kubectl set image replicaset my-replicaset nginx=nginx:1.21
kubectl describe pod my-replicaset-nlrcr
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
          command: ['sh', '-c', 'echo Hi!']
          env:
          - name: VAR_NAME
            value: var-value
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
```bash
kubectl get deployments
kubectl apply -f <file.yml>
kubectl set image deployment <name> '<containername/*=nginx:1.12'

kubectl rollout undo deployment <name> # откат к прошлой версии
kubectl rollout undo deployment <name> --to-revision=1  # на две версии назад

kubectl delete all --all
```
### ConfigMap
Передача в приложение настроек. Можно подключать к нескольким деплойментам.
Не нужно применять ничего, под раз в минуту сам смотрит и обновляет настройки (вольюм). 
```bash
kubectl apply -f <first.yml> -f <second.yml>
kubectl get cm

kubectl port-forward <container name> <host port>:<pod port> &
```
***
```yaml
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: my-configmap  
data:
  dbhost: postgresql
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

# внутри container на одном уровне с env
envFrom:
- configMapRef:
	name: my-configmap-env
```
### Secret
Как конфигмап, только информация хранится в закодированном виде. Для них можно определить права доступа.
Для применения секретов под нужно перезапустить.
- **generic** - пароли / токены для приложения
- **docker-registry** - данные для авторизации в докер
- **tls** - сертификаты для **ingress**
```bash
kubectl create secret generic <name> --from-literal=<key>=<value>
kubectl get secrets
kubectl get secrets test -o yaml

excho <value> | base64 -d
```
***
```yaml
env:
- name: TEST
  value: foo
- name: TEST_1
  valueFrom:
	secretKeyRef:
	  name: test
	  key: test1
```
***
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test
stringData:
  test: updated
```
### Downward API
Позволяет передавать параметры манифестов как переменные окружения или файлы.
Например, IP адрес / имя пода, полученный уже после применения манифеста.
***
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
        env:
        - name: TEST
          value: foo
        - name: TEST_1
          valueFrom:
            secretKeyRef:
              name: test
              key: test1
        - name: __POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: __POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: __POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        ports:
        - containerPort: 80
        volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
      volumes:
      - name: podinfo
        downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "annotations"
              fieldRef:
                fieldPath: metadata.annotations
```
### Service
- `_ClusterIP_` - для внутреннего использования
- `_NodePort_` - можно открыть порт в интернет (но неудобно, потому что не классические порты)
- `_LoadBalancer_` - используется только в облачных средах
- `_ExternalName_` - обращение к сервису по конкретному имени, которое можно задать
- `_ExternalIPs_` - как NodePort, только можно отдать прямо IP адреса
Объект кластера, который позволяет сделать статичный выход от многих эфимерных подов в один доступный сервис, на существование которого можно положиться. То есть поды могут перезапускаться, удаляться, а сервис - нет.
Это не сервис, а лишь правила IPTables!!
При создании автоматически создается объект класса **Endpoints**, в котором прописаны IP-адреса подходящих подов. 
Адрес сервиса также может меняться, поэтому можно отправлять запросы по его имени.
`Headless Service` - имеет ClusterIP: None, из-за чего можно настроить доступ к конкретным подам, а не Round-Robin. Удобно при репликации.
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
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: my-app
  type: ClusterIP
  
# or

apiVersion: v1
kind: Service
metadata:
  name: my-service-np
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: my-app
  type: NodePort
```
### Ingress
Проксирование запросов снаружи сети кубера внуть в сервис и дальше в конкретные поды
```bash
minikube addons enable ingress
minikube tunnel
```
***
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress-nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: my.s081569.edu.slurm.io  # домен, который нужно обслуживать
    http:
      paths:
      - pathType: Prefix
        path: "/"
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
### Volumes
`HostPath` - замонтировать путь из машины (ноды).
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
	app: my-app
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
        volumeMounts:
        - name: data
          mountPath: /files
      volumes:
      - name: data
        hostPath:
          path: /data_pod
```
***
`EmptyDir` - временный диск, который создается самим кубернетесом (существует только во время существования пода).
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
	app: my-app
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
        volumeMounts:
        - name: data
          mountPath: /files
      volumes:
      - name: data
        emptyDir: {}
```
***
`PV/PVC`
Облачное хранилище, или что-то другое
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: fileshare
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
И
```yaml
containers:
  - image: centosadmin/reloadable-nginx:1.12
	name: nginx
	ports:
  - containerPort: 80
	resources:
	  requests:
		cpu: 50m
		memory: 100Mi
	  limits:
		cpu: 100m
		memory: 100Mi
	volumeMounts:
	- name: config
	  mountPath: /etc/nginx/conf.d
	- name: data
	  mountPath: /data
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: fileshare 
```
### Probes
`Liveness` - работает ли (перезапуск, если нет)
`Readiness` - готово ли принимать трафик (убирается из балансировки)
`Startup` - проверка, запустилось ли приложение (только при старте)
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
          terminationMessagePolicy: FallbackToLogsOnError  # дает возможность посмотреть логи упавшего контейнера в текущем в describe
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
### Jobs
Создает под для выполнения задачи один раз.
`restartPolicy` - для контейнеров, лучше в `Never`, потому что иначе дело до `backoffLimit` даже не дойдет, потому что кублет будет постоянно пытаться перезапустить сам контейнер, а не под.
Джобы и созданные ими поды автоматически не удаляются. 
```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  ttlSecondsAfterFinished: 120  # после 2 минут независимо от статуса удаление
  backoffLimit: 2               # количество перезапусков в случае неудачи
  activeDeadlineSeconds: 60     # ожидание работы поды
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo Hello from the Kubernetes cluster
      restartPolicy: Never      # сами контейнеры не будут перезапускаться при ошибках
```
***
```yaml
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"    # каждую минуту
  concurrencyPolicy: Forbid  # лучше запретить параллельный запуск подов, но есть Allow
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 100
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: Never
```
### DaemonSet
**Статические поды** - поды, которые запускаются из конфигурации в `/etc/kubernetes/manifests` (например, `apiserver` или `etcd`).
- запускает поды на всех нодах сервера
- при добавлении ноды запускает на нем под
- при удалении ноды автоматически удаляется под (Garbage Collector)
- описание практически такое же, как и у `Deployment`
`Taints` - метка сервера-ноды:
- `NoSchedule` - запуск пользовательских под запрещен
- `NoExecute` - удалить даже существующие поды, когда такой эффект навешен
- `PreferNoSchedule` - "стараться" не размещать поды
`Tolerations` - разрешение на метку по `key`.
```bash
kubectl get nodes master-1.s081569.slurm.io -o json | jq '.spec.taints'
```
***
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: node-exporter
  name: node-exporter
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: k8s.gcr.io/pause:3.3
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: 10m
            memory: 64Mi
          requests:
            cpu: 10m
            memory: 64Mi
      nodeSelector:
        beta.kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      tolerations:
      - effect: NoSchedule
        operator: Exists  # игнорирование всех правил для NoSchedule
        # key: <key of taint> вместо operator
```
### Affinity
`.spec.affinity` - путь.
```yaml
affinity:  # определение расположения в кластере
  nodeAffinity: # или podAffinity, podAntiAffinity
	# учитывается при планировании, но игонировать, если уже запущено
	# если обязательно, меняем `preferred` -> `required`
	preferredDuringSchedulingIgnoredDuringExecution:
	  nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/e2e-az-name  # метка ноды
          operator: In
          values:                         # с такими значениями
          - e2e-az1  
          - e2e-az2
```
### StatefulSet
- позволяет запускать группу подов как Deployment
- при удалении не удаляет PVC, а создаются они из PVC Template
- нужно для запуска приложений с сохранением состояния (например, БД или очередь)
- у каждого пода постоянное и уникальное имя
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: rabbitmq
spec:
  serviceName: rabbitmq
  replicas: 2
  selector:
    matchLabels:
      app: rabbitmq
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      serviceAccountName: rabbitmq
      terminationGracePeriodSeconds: 10
      containers:
        - name: rabbitmq-k8s
          image: rabbitmq:3.7
          env:
            - name: MY_POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          ports:
            - name: amqp
              protocol: TCP
              containerPort: 5672
          resources:
            limits:
              cpu: 1000m
              memory: 1024Mi
            requests:
              cpu: 250m
              memory: 512Mi
          livenessProbe:
            exec:
              command: ["rabbitmqctl", "status"]
            initialDelaySeconds: 60
            periodSeconds: 60
            timeoutSeconds: 15
          readinessProbe:
            exec:
              command: ["rabbitmqctl", "status"]
            initialDelaySeconds: 20
            periodSeconds: 60
            timeoutSeconds: 10
          imagePullPolicy: Always
          volumeMounts:
            - name: config-volume
              mountPath: /etc/rabbitmq
            - name: data
              mountPath: /var/lib/rabbitmq
      volumes:
        - name: config-volume
          configMap:
            name: rabbitmq-config
            items:
              - key: rabbitmq.conf
                path: rabbitmq.conf
              - key: enabled_plugins
                path: enabled_plugins
      affinity:
	    # Anti - не то, где запустить, а то, где не запускать
        podAntiAffinity:
          # не обязательное правило, то есть если для реплик не будет хватать уникальныз нод - несколько могут работать на одной
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  # не запускаем поды на нодах, где уже есть поды с такими метками - т.е. разносим по разным нодам
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - rabbitmq
                # определение, на каких нодах запускать
                topologyKey: kubernetes.io/hostname
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```