------------------------------------------------------------------------------------
Lab 4: Working with volume mounts in Docker
------------------------------------------------------------------------------------

### Task 1: Starting Docker Containers Bind Mounts

```sh
mkdir /home/ubuntu/share
```
```sh
echo 'Hello From Docker Host' > /home/ubuntu/share/index.html
```
```sh
docker run -it --name container1 -p 80:80 -v /home/ubuntu/share:/var/www/html ubuntu:18.04 /bin/bash
```
```sh
apt-get update -y
```
```sh
apt-get install apache2 -y
```
```sh
service apache2 start
```
```sh
service apache2 status
```
```sh
echo 'Hello From Container1' > /var/www/html/index.html
```
Press `Ctrl+P+Q`, to switch back to Host

```sh
docker run -it --name container2 -v /home/ubuntu/share:/var/www/html ubuntu:18.04
```
```sh
echo 'Hello From Container2' > /var/www/html/index.html
```
```sh
exit
```
```sh
docker ps -a
```
```sh
docker rm -vf container1 container2
```

### Task 2: Create a bind mount with --mount option and verify it

```sh
docker run -d -it --name newbind01 --mount type=bind,source=/home/ubuntu/share/,target=/app nginx:latest
```
```sh
docker inspect newbind01
```
```sh
mount | grep -i /app
```

------------------------------------------------------------------------------------
Lab 5: Volume Mounting with Docker Containers
------------------------------------------------------------------------------------ 

### Task 1: Creating a new docker volume and inspecting containers

```sh
docker volume create ct-volume1
```
```sh
docker volume ls
```
```sh
docker volume inspect ct-volume1
```

### Task 2: Launching a Nginx container mapped to a specific docker volume and verification

```sh
docker run -d -p 80:80 --name=nginx-container --mount src=ct-volume1,dst=/usr/share/nginx/html nginx
```
```sh
docker ps
```
```sh
docker container inspect nginx-container
```
```sh
ls /var/lib/docker/volumes/ct-volume1/_data/
```
```sh
cd /var/lib/docker/volumes/ct-volume1/_data/
```
```sh
touch f1 f2 f3
```
```sh
vi /var/lib/docker/volumes/ct-volume1/_data/index.html
```

### Task 3: Deleting container and attaching the volume to another container

```sh
docker container stop nginx-container
```
```sh
docker rm container nginx-container
```
```sh
docker ps -a
```
```sh
docker run -it --name busybox-container1 --mount source=ct-volume1,target=/data busybox sh
```
```sh
ls
```
```sh
cd /data
```
```sh
ls
```
```sh
exit
```
```sh
docker stop busybox-container1
```
```sh
docker rm busybox-container1
```
```sh
docker ps -a
```
```sh
docker volume ls
```
```sh
docker volume rm ct-volume1
```
```sh
docker volume ls
```

### Task 4: Create a container with tmpfs mount and verify it

```sh
docker run -d -it --name tmpmount --mount type=tmpfs,destination=/app nginx:latest
```
```sh
docker container inspect tmpmount
```
