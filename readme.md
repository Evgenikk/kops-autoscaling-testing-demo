
## Deploy cluster (kops)

Получаем необходимые переменные окружения:
```
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
export KOPS_STATE_STORE=s3://kops.us-east-2.test
```

Создаем из шаблона конфигурацию кластера:
```
kops toolbox template --template cluster.tmpl.yaml --values us-2.values.yaml --output us-east-2.config.yaml
```

Применяем конфигурацию кластера, создаем секрет с публичным ключем:
```
kops create secret --name us-east-2.test.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub
kops replace -f us-east-2.config.yaml --force
```

Ожидаемый вывод:
```
I0331 19:32:01.390065   30068 replace.go:179] instanceGroup: bastions was not found, creating resource now
I0331 19:32:02.087053   30068 replace.go:179] instanceGroup: master.a was not found, creating resource now
I0331 19:32:02.741598   30068 replace.go:179] instanceGroup: master.b was not found, creating resource now
I0331 19:32:03.426033   30068 replace.go:179] instanceGroup: master.c was not found, creating resource now
I0331 19:32:04.134346   30068 replace.go:179] instanceGroup: nodes was not found, creating resource now
```

Применяем

```
kops update cluster us-east-2.test.k8s.local
```

Смотрим output, после чего добавляем флаг --yes:
```
kops update cluster us-east-2.test.k8s.local --yes
```
Ждем некоторое время и смотрим версию кластера:
```
kubectl get nodes 
```

## Cluster autoscaler

Устанавливаем автоскейлер:
```
kubectl apply -f cluster-autoscaler.yaml       
```

Показываем деплоймент, после чего применяем его, показываем сколько нод:
```
kubectl apply -f immense_pods.yaml 
```

Скейлим деплоймент
```
kubectl scale deployment immense --replicas=5    
```

Когда поднимется еще одна нода можно будет обратно отскейлить кластер:
```
kubectl scale deployment immense --replicas=1
```


## Sonoboy

Видим, что end-to-end занимают много времени:
```
sonobuoy run
sonobuoy status  
```

Смотрим результат:
```
 sonobuoy results sonar-result.tar.gz  --mode=dump  
```

## Polaris

```
kubectl apply -f https://github.com/fairwindsops/polaris/releases/latest/download/dashboard.yaml
kubectl port-forward --namespace polaris svc/polaris-dashboard 8080:80
```

##  Clean

```
rm -rf ./us-east-2.config.yaml
kops delete cluster us-east-2.test.k8s.local --yes
```