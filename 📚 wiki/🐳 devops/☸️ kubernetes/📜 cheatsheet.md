##### Создание, применение, изменение
```bash
kubectl create -f <yaml>
kubectl apply -f <yaml>
kubectl run --image <image:tag> <name> <command>

kubectl scale <deployment|replicaset> <name> --replicas=3

kubectl rollout undo deployment <name> --to-revision=1  # две версии тут

kubectl port-forward <container name> <host port>:<pod port>

kubectl create secret generic <name> --from-literal=<key>=<value>

kubectl delete <pod|replicas|deploy|endpoints|service> <name>
kubectl delete -f <yaml>

kubectl cp <namespace>/<pod>:<path> <path>
```
##### Получение информации
```bash
kubectl exec -ti <pod name> <command>
kubectl exec -ti <container> -- <command>

kubectl logs <pod name>

kubectl get <pod|replicas|deploy|endpoints|service> -o wide -n <namespace>

kubectl describe <pod|replicas|deploy|endpoints|service> <name>
```
##### 