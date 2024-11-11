# Домашнее задание к занятию "`Запуск приложений в K8S`" - `Макаров Денис`
---

## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

### 1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
Созданный Deployment и выполение команд  представлено на скриншотах ниже:

![Deployment](https://github.com/user-attachments/assets/28e1185a-1a23-4698-8db7-17cccd432762)

![pod deployment](https://github.com/user-attachments/assets/eb9ba5b8-589c-4794-946a-2f37c945081f)

![Error](https://github.com/user-attachments/assets/0a545362-06f9-413e-b10f-7439fffdc5c6)

Возникает ошибка, проверим логи, чтобы узнать причину:

![logs](https://github.com/user-attachments/assets/c0ea7b6c-741e-4457-91a8-18317673c0dc)

В Логах мы видим что в контейнере ```multitool``` еще одна копия nginx пытается занять 80 порт, который уже занят nginx из другого контейнера. Работа двух веб серверов с типовыми конфигурациями в одном поде невозможна. Решим проблему через изменение параметра HTTP_PORT у контейнера multitool:

![izm deployment](https://github.com/user-attachments/assets/74567f9d-a991-4b6b-a53f-25c7ac0ac0ed)


Проверим результат:

![proverka](https://github.com/user-attachments/assets/e8e1e252-def4-4241-bec7-d5482c85785c)

Ошибка больше не появляется, контейнеры работают

### 2. После запуска увеличить количество реплик работающего приложения до 2.

![replica](https://github.com/user-attachments/assets/05c1c2e6-4ca0-48d8-8a20-7e876b313d29)

### 3. Продемонстрировать количество подов до и после масштабирования.

![replica](https://github.com/user-attachments/assets/e536204f-9522-425f-93e4-0fae484e43c7)

### 4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
Созданный сервис и выполнение представлено на скриншотах ниже:

![service_nginx](https://github.com/user-attachments/assets/4bf5e661-bb70-41ac-803c-2459c378196c)

![proverka srv](https://github.com/user-attachments/assets/2c54de75-8e6b-4655-8e46-e16cdf10f167)

### 5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.
Созданный Pod с приложением multitool/, а также выполнение представлено на скриншотах ниже:


![test_pod](https://github.com/user-attachments/assets/24dd3c16-5ac0-4860-a770-0ebf302f5d6e)

![zapusk multitool](https://github.com/user-attachments/assets/0af1b5b8-f4f5-47b9-836f-e72969278021)

Проверим доступ до приложений:

![curl_multi test1](https://github.com/user-attachments/assets/09e9a127-fc26-4a51-bb32-7f4882f78868)


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
