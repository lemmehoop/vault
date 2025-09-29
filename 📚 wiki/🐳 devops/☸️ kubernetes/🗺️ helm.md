Основная проблема других инструментов - нет централизованного откатывания приложения со всеми манифестами (а не только `rollout` деплоймента), тогда как `helm` его предлагает: то есть и шаблонизация приложения, и отдельный инструмент для управления версиями всех манифестов.
- пакетный менеджер для релизов
- декларативность
- continuous delivery
	- `watch` - если что-то не так с релизом, то он сам попробует починить или откатить
	- `rollback` - откат при неудачном новом релизе
	- `hooks` - умеет делать что-то до или после релиза
- множество плагинов

### Chart
Пакет == чарт
`.tgz` состоит из:
- шаблонизированные манифесты
- файл со значениями переменных, которые используются в манифестах
- метаинформация (автор, репозиторий и прочее)

### Commands
```bash
helm search <chart>  # поиск
helm install <chart> # установка
helm upgrade <chart> # обновление
helm get <chart>     # скачивание
helm show            # информация о чарте
helm list            # список установленных чартов
helm uninstall       # удалить релиз
```

***

### Practice
```bash
helm repo add southbridge https://charts.southbridge.ru
helm repo update

helm search hub kube-ops
helm show values southbridge/kube-ops-view > values.yaml

helm install ops-view southbridge/kube-ops-view -f values.yaml

helm pull southbridge/kube-ops-view
tar -zxvf kube-ops-view-X.Y.Z.tgz

helm create <char name>
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}-{{ .Release.Name }}
  labels:
    app: {{ .Chart.Name }}-{{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
{{ if .Values.annotations }}
  annotations:
{{ toYaml .Values.annotations | indent 4 }}
{{ end -}}
spec:
  replicas: {{ .Values.replicas | default 2 }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}-{{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}-{{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: {{ .Values.image.name }}:{{ .Values.image.tag }}
        ports:
        - containerPort: {{ .Values.port }}
{{ if .Values.env }}
        env:
        {{- range $key, $val := .Values.env }}
        - name: {{ $key | quote }}
          value: {{ $val | quote }}
        {{- end }}
{{ end }}
        resources:
{{ toYaml .Values.resources | indent 10 -}}
```