# [OPS-949]
## Развернуть MySQL в kubernetes
### 1. Создаем манифесты для деплоя Mysql как "StatefulSet":

```
mysql-pv.yaml - Выделение места на ноде back
mysql-pvc.yaml - Запрос на выделения места
mysql-secrets.yaml - Создания секретов для Mysql
mysql-deploy.yaml - Деплой Mysql
mysql-service.yaml - Создание сервиса для подключения к Mysql
```

### 2. Запускаем манифесты:

```
mysql-pv.yaml - Выделение места на ноде back
mysql-pvc.yaml - Запрос на выделения места
mysql-secrets.yaml - Создания секретов для Mysql
mysql-deploy.yaml - Деплой Mysql
mysql-service.yaml - Создание сервиса для подключения к Mysql
```


!!!!!!!!!
Сертификаты для registry кладем в (именно CA)
/etc/docker/certs.d/registry/ca.crt
так же занести в /ets/hosts
ip_k8s_master registry
рестарт сервиса docker