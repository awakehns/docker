# Домашнее задание к занятию 4 `«Оркестрация группой Docker контейнеров на примере Docker Compose»` - `Демин Герман`

### Задание 1

Установка docker и docker compose:

```
sudo dnf update -y

sudo dnf install -y dnf-plugins-core -y

sudo nano /etc/yum.repos.d/docker-ce.repo
```

```
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/fedora/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/fedora/gpg
```

```
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo systemctl enable --now docker

sudo usermod -aG docker $USER

newgrp docker

sudo nano /etc/docker/daemon.json
```

```
{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://daocloud.io",
    "https://c.163.com/",
    "https://registry.docker-cn.com"
  ]
}
```

    sudo systemctl restart docker


Создание docker-образа
```
mkdir custom-nginx

cd custom-nginx

nano index.html
```

```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```

    nano Dockerfile


```
FROM nginx:1.21.1
COPY index.html /usr/share/nginx/html/index.html
```

Билд и пуш docker-образа
```
docker build -t awakehns/custom-nginx:1.0.0 .

docket login

docker push awakehns/custom-nginx:1.0.0
```

Успешный пуш докер образа:

![docker-push](/img/docker-push.png)

https://hub.docker.com/r/awakehns/custom-nginx



### Задание 2

Запуск и переименовывание контейнера

```
docker run -d --name "DeminGerman-custom-nginx-t2" -p 127.0.0.1:8080:80 awakehns/custom-nginx:1.0.0

docker rename DeminGerman-custom-nginx-t2 custom-nginx-t2

date +"%d-%m-%Y %T.%N %Z" ; sleep 0.150 ; docker ps ; ss -tlpn | grep 8080 ; docker logs custom-nginx-t2 -n1 ; docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html

curl http://127.0.0.1:8080
```

Успешный старт и переименовывание контейнера:

![docker-start](/img/docker-start.png)



### Задание 3

```
docker attach custom-nginx-t2

ctl+C

docker ps -a

```

Контейнер остановился, потому что завершился основной процесс (nginx в режиме fg)

Повторный запуск контейнера

    docker start custom-nginx-t2


Подключение к контейнеру, остановка, повторный запуск

![docker-attach-1](/img/docker-attach-1.png)


```
apt-get update && apt-get install -y nano

nano /etc/nginx/conf.d/default.conf
```

Меняем порт с 80 на 81

    docker exec -it custom-nginx-t2 bash




```
nginx -s reload

curl http://127.0.0.1:80
```

    curl: (7) Failed to connect to 127.0.0.1 port 80: Connection refused
    
```
curl http://127.0.0.1:81

exit
```

```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```

Тесты поллючения после смены порта

![docker-attach-2](/img/docker-attach-2.png)

    ss -tlpn | grep 8080
    
    LISTEN 0      4096            127.0.0.1:8080       0.0.0.0:*  
    
    docker port custom-nginx-t2
    
    80/tcp -> 127.0.0.1:8080

    curl http://127.0.0.1:8080
    
    curl: (56) Recv failure: Соединение разорвано другой стороной

Проверка проброшенных портов докера и тест подключения по старому порту

![docker-attach-3](/img/docker-attach-3.png)

Nginx теперь слушает порт 81, а контейнер проброшен на порт 80



### Задание 4

```
docker run -dit --name centos-container -v $(pwd):/data centos:7 bash

docker run -dit --name debian-container -v $(pwd):/data debian:bookworm bash

docker exec centos-v bash -c "echo 'From CentOS' > /data/centos.txt"

docker exec centos-container bash -c "echo 'From CentOS' > /data/centos.txt"

echo "From Host" > host.txt

docker exec debian-container ls -l /data

docker exec debian-container cat /data/centos.txt

docker exec debian-container cat /data/host.txt
```

Работа с файловой системой хоста и выполнение команд из контейнеров

![docker-4](/img/docker-4.png)



### Задание 5

Создание compose и локального registry
```
mkdir -p /tmp/netology/docker/task5

cd /tmp/netology/docker/task5

nano compose.yaml
```

```
version: "3"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

```
docker compose up -d

nano compose.yaml
```

```
version: "3.9"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
  registry:
    image: registry:2
    ports:
      - "5000:5000"
```

```
docker tag awakehns/custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx:latest

docker push 127.0.0.1:5000/custom-nginx:latest
```

Вывод результата запуска docker compose

![compose-1](/img/compose-1.png)

![compose-2](/img/compose-2.png)

Скриншот конфига из портейнера

![portainer](/img/portainer.png)

```
rm compose.yaml

docker compose up -d
```

Предупреждение говорит о том, что docker compose не нашёл compose.yaml, рекомендует переименовать docker-compose.yaml или использовать -f

Исправляем

    docker compose -f docker-compose.yaml up -d
    
Гасим compose одной командой

    docker compose down
    
Скриншот вывода команд

![down](/img/down.png)
