# Домашнее задание к занятию "`Запуск приложений в K8S`" - `Макаров Денис`



---

## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

### 1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
      - name: multitool
        image: wbitt/network-multitool 
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f my_deployment.yaml 
deployment.apps/netology-deployment created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS   RESTARTS     AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-74c89c477b-zz8k8   1/2     Error    1 (5s ago)   10s   10.1.123.144   netology-01   <none>           <none>
```
Возникает ошибка, проверим логи, чтобы узнать причину:
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl logs netology-deployment-74c89c477b-zz8k8 nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl logs netology-deployment-74c89c477b-zz8k8 multitool
The directory /usr/share/nginx/html is not mounted.
Therefore, over-writing the default index.html file with some useful information:
WBITT Network MultiTool (with NGINX) - netology-deployment-74c89c477b-zz8k8 - 10.1.123.144 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
2024/02/12 18:48:32 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)
2024/02/12 18:48:32 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)
2024/02/12 18:48:32 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)
2024/02/12 18:48:32 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)
2024/02/12 18:48:32 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)
2024/02/12 18:48:32 [emerg] 1#1: still could not bind()
nginx: [emerg] still could not bind()
```
Логи показывают, что в контейнере ```multitool``` еще одна копия nginx пытается занять 80 порт, который уже занят nginx из другого контейнера. Работа двух веб серверов с типовыми конфигурациями в одном поде невозможна. Решим проблему через изменение параметра HTTP_PORT у контейнера multitool:
```yaml
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "81"
```
Проверим результат:
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f my_deployment.yaml 
deployment.apps/netology-deployment configured
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          8s    10.1.123.145   netology-01   <none>           <none>
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl logs netology-deployment-5d89c69f57-jrwhc nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl logs netology-deployment-5d89c69f57-jrwhc multitool
The directory /usr/share/nginx/html is not mounted.
Therefore, over-writing the default index.html file with some useful information:
WBITT Network MultiTool (with NGINX) - netology-deployment-5d89c69f57-jrwhc - 10.1.123.145 - HTTP: 81 , HTTPS: 443 . (Formerly praqma/network-multitool)
Replacing default HTTP port (80) with the value specified by the user - (HTTP_PORT: 81).
```
Ошибка больше не появляется, контейнеры работают

### 2. После запуска увеличить количество реплик работающего приложения до 2.
Для решения вопроса заменим значение ```replicas``` на 2 и обновим deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  labels:
    app: main
spec:
  replicas: 2
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "81"
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f my_deployment.yaml 
deployment.apps/netology-deployment configured
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          3m43s   10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          6s      10.1.123.146   netology-01   <none>           <none>
```
### 3. Продемонстрировать количество подов до и после масштабирования.
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          8s    10.1.123.145   netology-01   <none>           <none>
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          3m43s   10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          6s      10.1.123.146   netology-01   <none>           <none>
```
### 4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: deployment-service
spec:
  ports:
    - name: http-app
      port:  80
      protocol: TCP
      targetPort: 80
    - name: https-app
      port:  443
      protocol: TCP
      targetPort: 443
    - name: http-app-mult
      port:  81
      protocol: TCP
      targetPort: 81
  selector:
    app: main
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                 AGE     SELECTOR
kubernetes           ClusterIP   10.152.183.1    <none>        443/TCP                 150m    <none>
deployment-service   ClusterIP   10.152.183.77   <none>        80/TCP,443/TCP,81/TCP   6m35s   app=main
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get ep -o wide
NAME                 ENDPOINTS                                                       AGE
kubernetes           192.168.101.26:16443                                            150m
deployment-service   10.1.123.145:443,10.1.123.146:443,10.1.123.145:80 + 3 more...   6m49s

```
### 5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-multitool
spec:
  containers:
    - image: wbitt/network-multitool
      name: test-multitool
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f test_multitool.yaml 
pod/test-multitool created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          27m   10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          23m   10.1.123.146   netology-01   <none>           <none>
test-multitool                         1/1     Running   0          12s   10.1.123.147   netology-01   <none>           <none>
```
Проверим доступ до приложений:
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl exec test-multitool -- curl deployment-service:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   175k      0 --:--:-- --:--:-- --:--:--  199k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl exec test-multitool -- curl deployment-service:443
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   255  100   <html> 0     0      0      0 --:--:-- --:--:-- --:--:--     0
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
<hr><center>nginx/1.24.0</center>
</body>
</html>
255    0     0  66458      0 --:--:-- --:--:-- --:--:-- 85000


root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl exec test-multitool -- curl deployment-service:81
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - netology-deployment-5d89c69f57-jrwhc - 10.1.123.145 - HTTP: 81 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   153  100   153    0     0  24849      0 --:--:-- --:--:-- --:--:-- 30600
```

------

## Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

### 1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-init
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-init
  template:
    metadata:
      labels:
        app: web-init
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.2
      initContainers:
      - name: init-busybox
        image: busybox
        command: ['sh', '-c', "until nslookup svc-nginx-init.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for svc-nginx-init; sleep 2; done"]
```
### 2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f nginx_init.yaml 
deployment.apps/nginx-init created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS     RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running    0          70m   10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running    0          66m   10.1.123.146   netology-01   <none>           <none>
test-multitool                         1/1     Running    0          43m   10.1.123.147   netology-01   <none>           <none>
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          5s    10.1.123.149   netology-01   <none>           <none>
```
Параллельно:
```bash
root@nikulin:~# kubectl get pods -w
NAME                                   READY   STATUS    RESTARTS   AGE
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          69m
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          66m
test-multitool                         1/1     Running   0          42m
nginx-init-5fbf9bd49c-stqwt            0/1     Pending   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Pending   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          1s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          3s
```
### 3. Создать и запустить Service. Убедиться, что Init запустился.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx-init
spec:
  ports:
    - name: web
      port: 80
  selector:
    app: web-init    
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f svc_nginx_init.yaml 
service/svc-nginx-init created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE    IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          71m    10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          68m    10.1.123.146   netology-01   <none>           <none>
test-multitool                         1/1     Running   0          44m    10.1.123.147   netology-01   <none>           <none>
nginx-init-5fbf9bd49c-stqwt            1/1     Running   0          109s   10.1.123.149   netology-01   <none>           <none>
```
Параллельно:
```bash
root@nikulin:~# kubectl get pods -w
NAME                                   READY   STATUS    RESTARTS   AGE
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          69m
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          66m
test-multitool                         1/1     Running   0          42m
nginx-init-5fbf9bd49c-stqwt            0/1     Pending   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Pending   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          0s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          1s
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          3s
nginx-init-5fbf9bd49c-stqwt            0/1     PodInitializing   0          105s
nginx-init-5fbf9bd49c-stqwt            1/1     Running           0          106s
```
### 4. Продемонстрировать состояние пода до и после запуска сервиса.
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f nginx_init.yaml 
deployment.apps/nginx-init created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS     RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running    0          70m   10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running    0          66m   10.1.123.146   netology-01   <none>           <none>
test-multitool                         1/1     Running    0          43m   10.1.123.147   netology-01   <none>           <none>
nginx-init-5fbf9bd49c-stqwt            0/1     Init:0/1   0          5s    10.1.123.149   netology-01   <none>           <none>root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                 AGE     SELECTOR
kubernetes           ClusterIP   10.152.183.1     <none>        443/TCP                 3h22m   <none>
deployment-service   ClusterIP   10.152.183.77    <none>        80/TCP,443/TCP,81/TCP   58m     app=main
```
```bash
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl apply -f svc_nginx_init.yaml 
service/svc-nginx-init created
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE    IP             NODE          NOMINATED NODE   READINESS GATES
netology-deployment-5d89c69f57-jrwhc   2/2     Running   0          71m    10.1.123.145   netology-01   <none>           <none>
netology-deployment-5d89c69f57-wqclk   2/2     Running   0          68m    10.1.123.146   netology-01   <none>           <none>
test-multitool                         1/1     Running   0          44m    10.1.123.147   netology-01   <none>           <none>
nginx-init-5fbf9bd49c-stqwt            1/1     Running   0          109s   10.1.123.149   netology-01   <none>           <none>
root@nikulin:/home/nikulinn/other/kuber_1-3/scr# kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                 AGE     SELECTOR
kubernetes           ClusterIP   10.152.183.1     <none>        443/TCP                 3h22m   <none>
deployment-service   ClusterIP   10.152.183.77    <none>        80/TCP,443/TCP,81/TCP   58m     app=main
svc-nginx-init       ClusterIP   10.152.183.172   <none>        80/TCP                  2m37s   app=web-init
```
