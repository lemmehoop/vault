В базовом случае использования `kubectl` авторизационные данные хранятся в конфиге и управляются с помощью команды `kubectl config`.
Настройки хранятся в `~/.kube/config` или могут быть переданы в команду с помощью флага `--config` или задав переменную окружения `KUBECONFIG`.
***
```bash
kubectl config set-cluster <cluster IP / DNS> ...

kubectl config set-credentials <username> --token <token>

kubectl config set-context <context name> --user <user> --cluster <cluster> --namespace <namespace or 'default'>

kubectl config use-context <context name>  # for example prod / test and etc.
```
***
`Role` - представляет собой лишь набор прав, которые на данный момент ещё не назначены никаким пользователям или субъектам, а лишь определяют, какие действия можно выполнять с определёнными ресурсами.
`RoleBinding` - связь конкретной роли с конкретным пользователем или группам пользователей. 
***
```yaml
---
# простая роль
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
  
---
# простая привязка роли
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane  # this should be ServiceAccount
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" is a Role or ClusterRole
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
***
```yaml
---
# service account for access (it generates token)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: user

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: user
roleRef:  # binding cluster role to view non-secret info in cluster
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects: # to our previously created user
  - kind: ServiceAccount
    name: user
    namespace: s081569
```
***
`ClusterRole` выдает права не только в рамках одного неймспейса, а в рамках всего кластера.
Если `RoleBinding` использует `ClusterRole`, то все равно права будут работать только в рамках конкретного кластера.
`ClusterRoleBinding` используется для привязки роли для всего кластера.
### Resource Quota
Устанавливает количество доступных ресурсов и объектов для неймспейса в кластере.
### Limit Ranges
Устанавливает ресурсы на объекты кластера: контейнеров, подов и PVC.
### Pod Security Policy
Контролирует аспекты безопасности в описании подов (например, может запрещать монтирование директорий с хостовой ноды).
Включается отдельно как плагин. 