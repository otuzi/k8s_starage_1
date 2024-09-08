# Домашнее задание к занятию «Хранение в K8s. Часть 1»

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.


### Ответ 
deployment для запуска контейнера busybox и multitool с volume с типом hostPath: 
busybox (busybox_deploy.yaml): 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox-app
  template:
    metadata:
      labels:
        app: busybox-app
    spec:
      containers:
        - name: busybox
          image: busybox
          command: ["/bin/sh"]
          args:
            - -c
            - >
              while true; do
                echo "$(date)" >> /shared/data.txt;
                sleep 5;
              done
          volumeMounts:
            - name: shared-storage
              mountPath: /shared
      volumes:
        - name: shared-storage
          hostPath:
            path: /mnt/data
            type: DirectoryOrCreate
```
multitool (multitool_deploy.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multitool-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multitool-app
  template:
    metadata:
      labels:
        app: multitool-app
    spec:
      containers:
        - name: multitool
          image: praqma/network-multitool
          command: ["/bin/sh"]
          args:
            - -c
            - >
              while true; do
                clear;
                cat /shared/data.txt;
                sleep 5;
              done
          volumeMounts:
            - name: shared-storage
              mountPath: /shared
      volumes:
        - name: shared-storage
          hostPath:
            path: /mnt/data
            type: DirectoryOrCreate
```

Запустим ранее созданный Deployment:
```bash
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl apply -f busybox_deploy.yaml 
deployment.apps/busybox-deployment created
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl apply -f multitool_deploy.yaml 
deployment.apps/multitool-deployment created
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl get pod 
NAME                                  READY   STATUS    RESTARTS   AGE
busybox-deployment-6f9bfb669f-j6d6v   1/1     Running   0          17s
multitool-deployment-d9d9764b-sscl5   1/1     Running   0          9s
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
```

подключимся к контейнеру multitool и увидем что он может прочитатать то что пишет в файл контейнер busybox:
```bash
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl exec -it multitool-deployment-d9d9764b-sscl5 -c multitool -- /bin/bash
bash-5.1# 
bash-5.1# 
bash-5.1# ls -la 
total 76
drwxr-xr-x    1 root     root          4096 Sep  8 09:40 .
drwxr-xr-x    1 root     root          4096 Sep  8 09:40 ..
drwxr-xr-x    1 root     root          4096 Jan  5  2022 bin
drwx------    2 root     root          4096 Jan  5  2022 certs
drwxr-xr-x    5 root     root           360 Sep  8 09:40 dev
drwxr-xr-x    1 root     root          4096 Jan  5  2022 docker
drwxr-xr-x    1 root     root          4096 Sep  8 09:40 etc
drwxr-xr-x    2 root     root          4096 Nov 12  2021 home
drwxr-xr-x    1 root     root          4096 Jan  5  2022 lib
drwxr-xr-x    5 root     root          4096 Nov 12  2021 media
drwxr-xr-x    2 root     root          4096 Nov 12  2021 mnt
drwxr-xr-x    2 root     root          4096 Nov 12  2021 opt
dr-xr-xr-x  221 root     root             0 Sep  8 09:40 proc
drwx------    1 root     root          4096 Jan  5  2022 root
drwxr-xr-x    1 root     root          4096 Sep  8 09:40 run
drwxr-xr-x    1 root     root          4096 Jan  5  2022 sbin
drwxr-xr-x    2 root     root          4096 Sep  8 09:40 shared
drwxr-xr-x    2 root     root          4096 Nov 12  2021 srv
dr-xr-xr-x   13 root     root             0 Sep  8 09:40 sys
drwxrwxrwt    2 root     root          4096 Nov 12  2021 tmp
drwxr-xr-x    1 root     root          4096 Jan  5  2022 usr
drwxr-xr-x    1 root     root          4096 Jan  5  2022 var
bash-5.1# 
bash-5.1# 
bash-5.1# cd shared/
bash-5.1# ls -la 
total 12
drwxr-xr-x    2 root     root          4096 Sep  8 09:40 .
drwxr-xr-x    1 root     root          4096 Sep  8 09:40 ..
-rw-r--r--    1 root     root          1421 Sep  8 09:44 data.txt
bash-5.1# cat data.txt 
Sun Sep  8 09:40:00 UTC 2024
Sun Sep  8 09:40:05 UTC 2024
Sun Sep  8 09:40:10 UTC 2024
Sun Sep  8 09:40:15 UTC 2024
Sun Sep  8 09:40:20 UTC 2024
Sun Sep  8 09:40:25 UTC 2024
Sun Sep  8 09:40:30 UTC 2024
Sun Sep  8 09:40:35 UTC 2024
Sun Sep  8 09:40:40 UTC 2024
Sun Sep  8 09:40:45 UTC 2024
Sun Sep  8 09:40:50 UTC 2024
Sun Sep  8 09:40:55 UTC 2024
Sun Sep  8 09:41:00 UTC 2024
Sun Sep  8 09:41:05 UTC 2024
Sun Sep  8 09:41:10 UTC 2024
Sun Sep  8 09:41:15 UTC 2024
Sun Sep  8 09:41:20 UTC 2024
Sun Sep  8 09:41:25 UTC 2024
Sun Sep  8 09:41:30 UTC 2024
Sun Sep  8 09:41:35 UTC 2024
Sun Sep  8 09:41:40 UTC 2024
Sun Sep  8 09:41:45 UTC 2024
Sun Sep  8 09:41:50 UTC 2024
Sun Sep  8 09:41:55 UTC 2024
Sun Sep  8 09:42:00 UTC 2024
Sun Sep  8 09:42:05 UTC 2024
Sun Sep  8 09:42:10 UTC 2024
Sun Sep  8 09:42:15 UTC 2024
Sun Sep  8 09:42:20 UTC 2024
Sun Sep  8 09:42:25 UTC 2024
Sun Sep  8 09:42:30 UTC 2024
Sun Sep  8 09:42:35 UTC 2024
bash-5.1# exit
exit
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
```
------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

### Ответ:
Требуемый файл DaemonSet (multitool-daemonset.yaml):

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: multitool-daemonset
spec:
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
        - name: multitool
          image: praqma/network-multitool
          command: ["/bin/sh"]
          args:
            - -c
            - >
              while true; do
                clear;
                cat /host-logs/syslog;
                sleep 10;
              done
          volumeMounts:
            - name: log-volume
              mountPath: /host-logs
              readOnly: true
      volumes:
        - name: log-volume
          hostPath:
            path: /var/log
            type: Directory
```

Ввод команд на ноде k8s и вывод последних 10 строк из файла syslog:
```bash
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
multitool-daemonset-rj2qf   1/1     Running   0          5m35s
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ kubectl exec -it multitool-daemonset-rj2qf -c multitool -- /bin/bash
bash-5.1# 
bash-5.1# cd host-logs/
bash-5.1# ls -la
total 1632
drwxrwxr-x   10 root     110           4096 Sep  8 09:26 .
drwxr-xr-x    1 root     root          4096 Sep  8 09:57 ..
-rw-r--r--    1 root     root          6676 Sep  8 09:24 alternatives.log
drwxr-xr-x    2 root     root          4096 Sep  8 09:24 apt
-rw-r-----    1 104      adm         186769 Sep  8 10:03 auth.log
-rw-r-----    1 104      adm           2497 Sep  8 09:23 auth.log.1
-rw-rw----    1 root     43          175488 Sep  8 10:03 btmp
-rw-rw----    1 root     43               0 Sep  8 09:22 btmp.1
-rw-r-----    1 root     adm           4996 Sep  8 09:23 cloud-init-output.log
-rw-r-----    1 104      adm         123885 Sep  8 09:23 cloud-init.log
drwxr-xr-x    2 root     root          4096 Sep  8 09:57 containers
drwxr-xr-x    2 root     root          4096 Jul 21  2020 dist-upgrade
-rw-r--r--    1 root     adm          72967 Sep  8 09:23 dmesg
-rw-r--r--    1 root     root          8663 Sep  8 09:24 dpkg.log
drwxr-xr-x    3 root     root          4096 Aug 31 18:48 installer
drwxr-sr-x    4 root     nginx         4096 Sep  8 09:22 journal
-rw-r-----    1 104      adm          12400 Sep  8 09:40 kern.log
-rw-r-----    1 104      adm          99643 Sep  8 09:23 kern.log.1
-rw-rw-r--    1 root     43          292292 Sep  8 09:37 lastlog
drwxr-x---    6 root     root          4096 Sep  8 09:57 pods
drwx------    2 root     root          4096 Jul 31  2020 private
-rw-r-----    1 104      adm         752744 Sep  8 10:03 syslog
-rw-r-----    1 104      adm         137051 Sep  8 09:23 syslog.1
drwxr-x---    2 root     adm           4096 Sep  8 09:23 unattended-upgrades
-rw-rw-r--    1 root     43            3072 Sep  8 09:37 wtmp
bash-5.1# tail -n 10 syslog
Sep  8 10:03:23 fhmrlj60b9llbp4hne8n systemd[712]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.1NzTuE.mount: Succeeded.
Sep  8 10:03:23 fhmrlj60b9llbp4hne8n systemd[1]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.1NzTuE.mount: Succeeded.
Sep  8 10:03:23 fhmrlj60b9llbp4hne8n systemd[1]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.Vu9C0L.mount: Succeeded.
Sep  8 10:03:23 fhmrlj60b9llbp4hne8n systemd[712]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.Vu9C0L.mount: Succeeded.
Sep  8 10:03:24 fhmrlj60b9llbp4hne8n systemd[1]: run-containerd-runc-k8s.io-14ce0c70b81523c331c4fd38037f3b9bb2c142c2d06b876a7e190bfdf57d4992-runc.xySOS3.mount: Succeeded.
Sep  8 10:03:24 fhmrlj60b9llbp4hne8n systemd[712]: run-containerd-runc-k8s.io-14ce0c70b81523c331c4fd38037f3b9bb2c142c2d06b876a7e190bfdf57d4992-runc.xySOS3.mount: Succeeded.
Sep  8 10:03:32 fhmrlj60b9llbp4hne8n systemd[712]: run-containerd-runc-k8s.io-14ce0c70b81523c331c4fd38037f3b9bb2c142c2d06b876a7e190bfdf57d4992-runc.StlV6M.mount: Succeeded.
Sep  8 10:03:32 fhmrlj60b9llbp4hne8n systemd[1]: run-containerd-runc-k8s.io-14ce0c70b81523c331c4fd38037f3b9bb2c142c2d06b876a7e190bfdf57d4992-runc.StlV6M.mount: Succeeded.
Sep  8 10:03:33 fhmrlj60b9llbp4hne8n systemd[1]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.PAXd4V.mount: Succeeded.
Sep  8 10:03:33 fhmrlj60b9llbp4hne8n systemd[712]: run-containerd-runc-k8s.io-628ce702d435f7e4ecd52426a758df5f2d76476dd1595931ac710218cb5688c7-runc.PAXd4V.mount: Succeeded.
bash-5.1# exit
exit
ubuntu@fhmrlj60b9llbp4hne8n:~/k8s$ 
```
