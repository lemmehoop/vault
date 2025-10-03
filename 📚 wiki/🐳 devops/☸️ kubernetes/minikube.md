```bash
cp ~/Downloads/<cert>.pem ~/.minikube/certs # добавить сертификаты в доверенные

minikube addons list                       # получить список плагинов
minikube addons enable <ingress/dashboard> # включить плагин
minikube dashboard                         # смотреть в дашборд

eval $(minikube docker-env)  # переключить докер на тот, что внутри куба
```