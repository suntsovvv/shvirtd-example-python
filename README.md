
### Задача 1
Dockerfile.python:  
```
FROM python:3.9-slim  
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
RUN python3 -m venv venv
COPY main.py ./
CMD ["python", "main.py"]
```

### Задача 2(*)
```
root@study:/git/shvirtd-example-python# docker build -f Dockerfile.python -t cr.yandex/crpmvidvusvhijpfejv1/web-python:1.0 .
```

```
root@study:/git/shvirtd-example-python# yc container image list
+----------------------+---------------------+---------------------------------+------+-----------------+
|          ID          |       CREATED       |              NAME               | TAGS | COMPRESSED SIZE |
+----------------------+---------------------+---------------------------------+------+-----------------+
| crpo250gpshlv3jtjc8p | 2024-01-26 02:57:05 | crpmvidvusvhijpfejv1/web-python |  1.0 | 95.9 MB         |
+----------------------+---------------------+---------------------------------+------+-----------------+

root@study:/git/shvirtd-example-python# yc container image scan crpo250gpshlv3jtjc8p
done (1m31s)
id: chen762u86ub164hrcun
image_id: crpo250gpshlv3jtjc8p
scanned_at: "2024-01-26T10:07:38.050Z"
status: READY
vulnerabilities:
  critical: "1"
  high: "4"
  medium: "22"
  low: "67"

```

### Задача 3  

compose.yaml:

```
version: "3"
include:
  - proxy.yaml
x-env_file: &env_file
  env_file:
    - .env
x-restart: &restart
  restart: always #no, on-failure , always(default), unless-stopped
services:
  db:
    image: mysql:8
    container_name: db
    networks:
      backend:
        ipv4_address: 172.20.0.10
    <<: [*env_file,*restart]
    environment:
      - MYSQL_ROOT_HOST="%"
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./db_data:/var/lib/mysql
    ports:
      - "3306"


  web:
    image: cr.yandex/crpmvidvusvhijpfejv1/web-python:1.0
    container_name: web
    depends_on: ["db"]
    <<: [*env_file,*restart]
    environment:
      - DB_HOST=${DB_HOST}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_USER=${DB_USER}
      - DB_NAME=${DB_NAME}
    volumes:
      - ./venv:/bin/activate
    networks:
      backend:
        ipv4_address: 172.20.0.5
    ports:
    - "5000"

```
.env:  

```
DB_HOST=172.20.0.10
DB_USER=app
DB_PASSWORD=very_strong
DB_NAME=example
DB_ROOT_PASSWORD=ifkrhfhfp
```  
```
root@study:/git/shvirtd-example-python# curl -L http://127.0.0.1:8080
TIME: 2024-01-26 10:23:49, IP: Noneroot@study:/git/shvirtd-example-python#
root@study:/git/shvirtd-example-python# curl -L http://127.0.0.1:8090
TIME: 2024-01-26 10:24:12, IP: 127.0.0.1root@study:/git/shvirtd-example-python#
```

```
root@study:/git/shvirtd-example-python# docker exec -it db mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| example            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use example;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_example |
+-------------------+
| requests          |
+-------------------+
1 row in set (0.00 sec)

mysql> show tables; SELECT * from requests LIMIT 10;
+-------------------+
| Tables_in_example |
+-------------------+
| requests          |
+-------------------+
1 row in set (0.00 sec)

+----+---------------------+-------------+
| id | request_date        | request_ip  |
+----+---------------------+-------------+
|  1 | 2024-01-26 08:26:19 | NULL        |
|  2 | 2024-01-26 08:26:33 | NULL        |
|  3 | 2024-01-26 08:26:40 | 127.0.0.1   |
|  4 | 2024-01-26 08:36:22 | 10.0.104.57 |
|  5 | 2024-01-26 08:43:18 | 127.0.0.1   |
|  6 | 2024-01-26 08:43:28 | NULL        |
|  7 | 2024-01-26 08:44:45 | 10.0.104.57 |
|  8 | 2024-01-26 08:47:27 | 10.0.104.32 |
|  9 | 2024-01-26 10:23:49 | NULL        |
| 10 | 2024-01-26 10:24:12 | 127.0.0.1   |
+----+---------------------+-------------+
10 rows in set (0.01 sec)

mysql>
```  
### Задача 4  
Сделал в скрипте простой ввод переменных в .env файл. С каждым запуском скрипта файл будет перезаписываться.  
К регистру на YC сделал свободный доступ на скачивание образов.
Bash скрипт:  
```
#!/bin/bash
apt update
cd /opt
git clone https://github.com/suntsovvv/shvirtd-example-python /opt/shvirtd-example-python
cd /opt/shvirtd-example-python
echo "Please enter DB_ROOT_PASSWORD "
read a
echo "DB_ROOT_PASSWORD=$a" > .env
echo "Please enter DB_USER "
read b
echo "DB_USER=$b" >> .env
echo "Please enter DB_PASSWORD "
read c
echo "DB_PASSWORD=$c" >> .env
echo "Please enter DB_NAME "
read d
echo "DB_NAME=$d" >> .env
echo "DB_HOST=172.20.0.10" >> .env
docker compose up -d
```
Запуск скрипта:
```
root@fv44f521bi541f1iqsqq:/home/user# sh start.sh
Hit:1 http://mirror.yandex.ru/ubuntu jammy InRelease
Hit:2 http://mirror.yandex.ru/ubuntu jammy-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu jammy-backports InRelease
Hit:4 https://download.docker.com/linux/ubuntu jammy InRelease
Hit:5 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
3 packages can be upgraded. Run 'apt list --upgradable' to see them.
Cloning into '/opt/shvirtd-example-python'...
remote: Enumerating objects: 205, done.
remote: Counting objects: 100% (176/176), done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 205 (delta 132), reused 164 (delta 127), pack-reused 29
Receiving objects: 100% (205/205), 5.19 MiB | 8.76 MiB/s, done.
Resolving deltas: 100% (133/133), done.
Please enter DB_ROOT_PASSWORD
ifkrhfhfp
Please enter DB_USER
user
Please enter DB_PASSWORD
pass
Please enter DB_NAME
name
[+] Running 35/14
 ✔ ingress-proxy 6 layers [⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                                              20.0s
 ✔ db 10 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                                                    33.7s
 ✔ web 10 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                                                   23.0s
 ✔ reverse-proxy 5 layers [⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                                                                               16.0s

[+] Running 4/5
 ⠏ Network shvirtd-example-python_backend            Created                                                                                                                                                                            5.9s
 ✔ Container db                                      Started                                                                                                                                                                            5.3s
 ✔ Container shvirtd-example-python-reverse-proxy-1  Started                                                                                                                                                                            5.3s
 ✔ Container shvirtd-example-python-ingress-proxy-1  Started                                                                                                                                                                            5.0s
 ✔ Container web                                     Started                                                                                                                                                                            2.0s
root@fv44f521bi541f1iqsqq:/home/user# docker ps
CONTAINER ID   IMAGE                                           COMMAND                  CREATED          STATUS                                  PORTS                                                    NAMES
779cb26bf427   cr.yandex/crpmvidvusvhijpfejv1/web-python:1.0   "python main.py"         11 seconds ago   Restarting (1) Less than a second ago                                                            web
1e5e4a8326d4   nginx:latest                                    "/docker-entrypoint.…"   14 seconds ago   Up 8 seconds                                                                                     shvirtd-example-python-ingress-proxy-1
57aaa6c91fb2   mysql:8                                         "docker-entrypoint.s…"   14 seconds ago   Up 8 seconds                            33060/tcp, 0.0.0.0:32768->3306/tcp, :::32768->3306/tcp   db
0e200862ad1d   haproxy                                         "docker-entrypoint.s…"   14 seconds ago   Up 8 seconds                            127.0.0.1:8080->8080/tcp                                 shvirtd-example-python-reverse-proxy-1
root@fv44f521bi541f1iqsqq:/home/user# docker exec db mysql -uroot -p
Enter password: ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
root@fv44f521bi541f1iqsqq:/home/user# docker exec -ti db mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| name               |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql> use name;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables; SELECT * from requests LIMIT 10;
+----------------+
| Tables_in_name |
+----------------+
| requests       |
+----------------+
1 row in set (0.00 sec)

+----+---------------------+-----------------+
| id | request_date        | request_ip      |
+----+---------------------+-----------------+
|  1 | 2024-01-26 15:23:16 | 167.235.135.184 |
|  2 | 2024-01-26 15:23:16 | 195.211.27.85   |
|  3 | 2024-01-26 15:23:16 | 185.209.161.169 |
|  4 | 2024-01-26 15:23:18 | 209.14.69.16    |
|  5 | 2024-01-26 15:23:18 | 38.47.52.199    |
|  6 | 2024-01-26 15:23:18 | 38.145.202.12   |
|  7 | 2024-01-26 15:23:18 | 172.105.170.45  |
|  8 | 2024-01-26 15:23:19 | 185.143.223.66  |
|  9 | 2024-01-26 15:23:19 | 50.114.207.110  |
| 10 | 2024-01-26 15:23:19 | 5.44.42.40      |
+----+---------------------+-----------------+
10 rows in set (0.00 sec)
```  
