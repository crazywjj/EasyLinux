



```
kubectl logs --tail 200 -f podname -n jenkins
kubectl describe pod kubernetes-dashboard-849cd79b75-s2snt --namespace kube-system
```



## 场景一：docker mysql

1. 导出

```text
docker exec -i mysql_container mysqldump -uroot -proot --databases database_name --skip-comments > /path/to/my/dump.sql
```

2. 导入

```text
docker cp /path/to/my/dump.sql mycontainer:/dump.sql
docker exec mysql_container /bin/bash -c 'mysql -uroot -proot < /dump.sql'
```

场景二：k8s，更简单一点

1. 导出

```text
kubectl exec mysql-58 -n sql -- mysqldump -u root -proot --all-databases > dump_all.sql
```

2. 导入

```text
kubectl exec -it mysql-58 -n sql -- mysql -u root -proot USERS < dump_all.sql
```





