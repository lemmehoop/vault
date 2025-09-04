##### База
`Helm` - пакетный менеджер для `Kubernetes`.
Packaging kubernetes YAML files and distributing them.

`Helm Charts` - пакет с YAML файлами.
То есть используя готовый чарт можно развернуть нужное приложение одной командой.

```bash
helm search <package name>  # поиск пакетов

# сборка и установка
helm package --version=0.0.1 chart-${PROJECT_NAME}
helm install -f Values.yaml chartname ./chart
helm upgrade --install --atomic <name> -f ./Values.yaml -f ./${ENV}-Values.yaml ./${PROJECT_NAME}-0.0.1.tgz -n ${NAMESPACE}
```

Имеет `Template Engine`, благодаря которому можно создавать шаблоны, а затем передавать туда значения, чтобы динамически менять чарты
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: {{ .Values.name }}
spec:
	containers:
		- name: {{ .Values.container.name }}
		- image: {{ .Values.container.image }}
```

Собрав все нужные файлы воедино, чаще всего создается свой чарт, которым деплоится приложение.
##### Структура
```
chart-<project>/
	Chart.yaml   # name, dependencies and version
	values.yaml  # values configuration for templates (with defaults)
	charts/      # dependencies on other charts
	templates/   # шаблоны для выполнения (манифесты кубера)
```
