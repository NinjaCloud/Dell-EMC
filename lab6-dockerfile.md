# Lab 7: Building a Dockerfile to Set Up an Ubuntu Container with WordPress

## Task 1: Deploying MySQL and WordPress Containers

### Step 1: Create a Project Directory

```sh
mkdir wordpress
cd wordpress
```

### Step 2: Create a Dockerfile

```sh
vi Dockerfile
```

### Step 3: Add the Following Content to Dockerfile

```dockerfile
FROM ubuntu:18.04
MAINTAINER ADMIN "admin@cloudthat.com"
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get -q -y install apache2 \
    php7.2 \
    php7.2-fpm \
    php7.2-mysql \
    libapache2-mod-php7.2

ADD http://wordpress.org/latest.tar.gz /tmp

RUN tar xzvf /tmp/latest.tar.gz -C /tmp  \
    && cp -R /tmp/wordpress/* /var/www/html

RUN rm /var/www/html/index.html && \
    chown -R www-data:www-data /var/www/html

EXPOSE 80
CMD ["/bin/bash","-c","service apache2 start && sleep 5000"]
```

### Step 4: (Optional) Download Dockerfile from S3

```sh
wget https://hpe-content.s3.ap-south-1.amazonaws.com/Dockerfile
```

### Step 5: Build the Docker Image

```sh
docker build -t ct-wordpress:v1 .
```

### Step 6: Verify the Image Build

```sh
docker image ls
```

### Step 7: Create a Docker Network

```sh
docker network create --driver bridge ct-bridge
```

### Step 8: Run a MySQL Container

```sh
docker run -d --network ct-bridge --name mysql \
    -e MYSQL_DATABASE=wordpress \
    -e MYSQL_USER=admin \
    -e MYSQL_PASSWORD=password \
    -e MYSQL_ROOT_PASSWORD=password \
    mysql:5.7
```

### Step 9: Verify Running Containers

```sh
docker ps
```

### Step 10: Run the WordPress Container

```sh
docker run -d --network ct-bridge -p 80:80 ct-wordpress:v1
```

### Step 11: Verify Running Containers Again

```sh
docker ps
