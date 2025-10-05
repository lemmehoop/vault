### Кубер в CI
Для этого нужно:
- создать неймспейс
- создать сервисный аккаунт для работы в кластере и НС
- создать роль, в которой будут нужные права для деплоя приложения
- создать RoleBinding, который свяжет сервисный аккаунт и роль
- получить токен доступа, и пихнуть его в GitLab CI Variables
	- `kubectl -n "$NS" create token "$SA"`
### Образ из registry в kubernetes
```bash
#!/bin/bash
NS=s081569-xpaste-production

kubectl delete secret xpaste-gitlab-registry --namespace "$NS"

kubectl create secret docker-registry xpaste-gitlab-registry \
  --docker-server '<docker registry address>' \
  --docker-email '<email address for registry>' \
  --docker-username '<credentials username>' \
  --docker-password '<credentials password>' \
  --namespace "$NS"
```