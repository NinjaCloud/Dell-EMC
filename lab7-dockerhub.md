# Lab 8: Pushing the Image to DockerHub

## Task: Pushing the Image to DockerHub

### Step 1: Log in to DockerHub

```sh
docker login
```

### Step 2: Tag the Image for DockerHub

```sh
docker tag ct-wordpress:v1 <your-dockerhub-username>/ct-wordpress:v1
```

### Step 3: Verify the Tagged Image

```sh
docker image ls
```

### Step 4: Push the Image to DockerHub

```sh
docker push <your-dockerhub-username>/ct-wordpress:v1
```

### Step 5: Verify Pushed Image

```sh
docker image ls
```

### Step 6: Remove Local Images to Test Pull

```sh
docker image rm <your-dockerhub-username>/ct-wordpress:v1 ct-wordpress:v1
```

### Step 7: Run the Image from DockerHub

```sh
docker run -d -p 8080:80 <your-dockerhub-username>/ct-wordpress:v1
```

### Step 8: Stop and Remove the Container

```sh
docker stop <container-id-or-name>
docker container rm <container-id-or-name>
```

### Step 9: Remove Unused Images

```sh
docker image ls
docker image rm <image-id-or-name> <image-id-or-name>
docker image ls
docker image ls -a
```
