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

Созданный deployment предмтавлен на скриншоте ниже:
![pod nginx_init](https://github.com/user-attachments/assets/a3c2ede8-8707-463a-84c1-ac00bf5a7af7)


### 2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.

![status](https://github.com/user-attachments/assets/823a115c-1c62-4196-b5ca-5344f5321885)

### 3. Создать и запустить Service. Убедиться, что Init запустился.

![service_nginx-init](https://github.com/user-attachments/assets/ebaceeaa-064b-4541-b435-b4915e0f5755)

![status](https://github.com/user-attachments/assets/0ad96dbe-6d57-4b70-ab0c-e4acf6df475a)

### 4. Продемонстрировать состояние пода до и после запуска сервиса.

![status](https://github.com/user-attachments/assets/29197e66-eda1-406c-94fc-f96c9cb6a1c8)

![final](https://github.com/user-attachments/assets/9df74d0a-2381-45c3-a2e5-14093b566c84)
