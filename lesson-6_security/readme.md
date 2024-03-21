```
kubectl create ns app
kubectl config set-context --current --namespace=app

# применяем простой под
kubectl apply -f pod.yaml

# показать приатаченный default sa
kubectl describe po app-kuber

# заходим в под
kubectl exec -it app-kuber -- /bin/bash

# показать серт, токен, можно jwt decode показать для токена
cd /var/run/secrets/kubernetes.io/serviceaccount
```
---
```
# задаем переменные 
KUBEAPI=https://kubernetes.default.svc
SA=/var/run/secrets/kubernetes.io/serviceaccount
CACERT=${SA}/ca.crt
TOKEN=$(cat ${SA}/token)
NAMESPACE=$(cat ${SA}/namespace)
```
---
```
# с произвольными кредами - Unauthorized
curl --cacert ${CACERT} --header "Authorization: Bearer gsdhfgsdhfgshdfghsdf" -X GET ${KUBEAPI}/api

# с норм кредами - нормальный доступ
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${KUBEAPI}/api

# но Forbidden для чтения чего либо, т.к нет прав
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${KUBEAPI}/api/v1/namespaces/${NAMESPACE}/pods
```
---
```
# выходим, удаляем под
exit
kubectl delete -f pod.yaml
```
---
```
# применяем sa role и rolebinding, можно разобрать структуру манифестов и доступные параметры и что они означают
kubectl apply -f sa.yaml -f role.yaml -f rb.yaml

#применяем под с подключенным sa
kubectl apply -f pod_sa.yaml

# заходим в под
kubectl exec -it app-kuber -- /bin/bash

# показать серт, токен, можно jwt decode показать для токена, что теперь это другой SA
cd /var/run/secrets/kubernetes.io/serviceaccount
```
---
```
# задаем переменные 
KUBEAPI=https://kubernetes.default.svc
SA=/var/run/secrets/kubernetes.io/serviceaccount
CACERT=${SA}/ca.crt
TOKEN=$(cat ${SA}/token)
NAMESPACE=$(cat ${SA}/namespace)
```
---
```
# теперь можно получить список под в namespace
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${KUBEAPI}/api/v1/namespaces/${NAMESPACE}/pods
```
