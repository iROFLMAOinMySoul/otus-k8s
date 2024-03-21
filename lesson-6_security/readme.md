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

# можно также посмотреть список деплоев, т.к давали права на них, также подергать другие ручки и увидеть что прав на них нет
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${KUBEAPI}/apis/apps/v1/namespaces/${NAMESPACE}/deployments
```
---
```
# создаем сертификат для нового пользователя, которому хотим предоставить доступ в кластер
# подключаемся на ноду миникуба
minikube ssh
# создаем директорию для сертификата и ключа
sudo mkdir -p /cert
# создаем сертификат
sudo openssl genrsa -out /cert/otus.pem 2048
sudo openssl req -new -key /cert/otus.pem -out /cert/otus-csr.pem -subj "/CN=otus/O=team1/"
sudo openssl x509 -req -in /cert/otus-csr.pem -CA /var/lib/minikube/certs/ca.crt -CAkey /var/lib/minikube/certs/ca.key -CAcreateserial -out /cert/otus.crt -days 10000
# копируем на нашу хостовую машину получившиеся файлы

# настраиваем контекст для нового пользователя
kubectl config set-credentials otus --client-certificate=#PATH_TO_CERT_FILE#/otus.crt --client-key=#PATH_TO_KEY_FILE#/otus.pem
kubectl config set-context otus --cluster=minikube --user otus
# переключаемся на этот контекст
kubectl config use-context otus

# убеждаемся что у нас нет прав ни на что
kubectl get all
# но при этом мы аутентифицированы
kubectl auth whoami

# идем обратно в контекст миникуба
kubectl config use-context minikube
# применяем cluster role binding дающий пользователям группы team1 роль view
kubectl apply -f crb.yaml
#снова переключаемся на контекст тестового пользователя
kubectl config use-context otus
#видим что права на чтение появились
kubectl get all
```
