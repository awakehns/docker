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

`sudo systemctl restart docker`


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

`nano Dockerfile`


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

`docker start custom-nginx-t2`


Подключение к контейнеру, остановка, повторный запуск

![docker-attach-1](/img/docker-attach-1.png)


```
apt-get update && apt-get install -y nano

nano /etc/nginx/conf.d/default.conf
```

Меняем порт с 80 на 81

`docker exec -it custom-nginx-t2 bash`




```
nginx -s reload

curl http://127.0.0.1:80
```

```
curl http://127.0.0.1:81

exit
```

Тесты подключения после смены порта

![docker-attach-2](/img/docker-attach-2.png)

```
ss -tlpn | grep 8080`
    
docker port custom-nginx-t2

curl http://127.0.0.1:8080
```

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

`docker compose -f docker-compose.yaml up -d`
    
Гасим compose одной командой

`docker compose down`
    
Скриншот вывода команд

![down](/img/down.png)

# Домашнее задание к занятию 5 `«Практическое применение Docker»` - `Демин Герман`

### Задание 1

```
git clone https://github.com/awakehns/shvirtd-example-python.git

cd shvirtd-example-python

nano Dockerfile.python
```

```
FROM python:3.12-slim

WORKDIR /app

# Копируем все файлы проекта в контейнер
COPY . .

# Устанавливаем зависимости из requirements.txt (если есть)
RUN pip install --no-cache-dir -r requirements.txt

# Запускаем приложение через uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000"]
```

`nano .dockerignore`

```
__pycache__/
*.pyc
*.pyo
*.pyd
*.log
*.env
*.sqlite3
.env
.dockerignore
.git
.gitignore
venv/
.idea/
.vscode/
```

```
docker build -t my-fastapi-app -f Dockerfile.python .

docker run -p 5000:5000 my-fastapi-app
```

Билд и запуск сборки

![2-build-and-run](/img/2-build-and-run.png)



### Задание 3

`nano compose.yaml`

```
include:
  - proxy.yaml

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.python
    # Или если образ уже в реестре Yandex Cloud, используйте:
    # image: cr.yandex/<ваш-регион>/<ваш-репозиторий>/webapp:latest
    restart: always
    networks:
      backend:
        ipv4_address: 172.20.0.5
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    expose:
      - "5000"

  db:
    image: mysql:8
    restart: always
    networks:
      backend:
        ipv4_address: 172.20.0.10
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:

networks:
  backend:
    external: true
```

`nano Dockerfile.python`

```
FROM python:3.10-slim

WORKDIR /app

COPY app.py .

RUN pip install flask mysql-connector-python

ENV FLASK_APP=app.py

CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

`nano app.py`

```
from flask import Flask, request
import socket
import datetime
import mysql.connector

app = Flask(__name__)

@app.route('/')
def index():
    # Подключение к БД
    conn = mysql.connector.connect(
        host='db',
        user='exampleuser',
        password='examplepass',
        database='example'
    )

    cursor = conn.cursor()
    # Создание таблицы при первом запуске
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS requests (
            id INT AUTO_INCREMENT PRIMARY KEY,
            time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            ip VARCHAR(45)
        );
    ''')

    # Вставка IP-адреса
    user_ip = request.remote_addr
    cursor.execute('INSERT INTO requests (ip) VALUES (%s)', (user_ip,))
    conn.commit()

    cursor.close()
    conn.close()

    return f"Time: {datetime.datetime.now()}<br>IP: {user_ip}\n"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```
docker compose up --build -d

curl -L http://127.0.0.1:8090

docker exec -ti shvirtd-example-python-db-1 mysql -uroot -psupersecretrootpass

show databases;

use example;

show tables;

SELECT * from requests LIMIT 10;

\q

docker compose down
```

curl, обращение к db, вывод таблицы, выход из db, остановка проекта

![curl](/img/curl.png)



Для следующего задания я буду использовать pacman вместо dnf, так как сменил дистрибутив

### Задание 6

```
docker pull hashicorp/terraform:latest

sudo pacman -S dive

mkdir terraform

cd terraform/

docker save hashicorp/terraform:latest -o terraform.tar.gz

tar xvf terraform.tar.gz 

mkdir extracted

find blobs/sha256 -type f | while read blob; do
    tar -xf "$blob" -C extracted 2>/dev/null
done

mkdir ~/terraform

cp extracted/bin/terraform ~/terraform

~/terraform/terraform version
```

Сохранение бинарника terraform из образа через docker save

![terraform-save](/img/terraform-save.png)



### Задание 6.1

```
docker create --name tf-temp hashicorp/terraform:latest

mkdir ~/terraform/cp

docker cp tf-temp:/bin/terraform ~/terraform/cp/

ls -l ~/terraform/cp/

~/terraform/cp/terraform version

docker rm tf-temp
```

Сохранение бинарника terraform из образа через docker cp

![terraform-cp](/img/terraform-cp.png)
