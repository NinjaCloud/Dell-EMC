------------------------------------------------------------------------------------
Lab 5: Docker Networking 
------------------------------------------------------------------------------------

Task 1: Create a new docker bridge and check connectivity between containers of same bridge

docker network ls
docker network create --driver bridge ct-bridge1
docker network inspect ct-bridge1
docker network ls
docker run -it --network ct-bridge1 --name=ct-c1 busybox
Press Ctrl+P+Q, to switch back to Host
docker run -it --network ct-bridge1 --name=ct-c2 busybox
Press Ctrl+P+Q, to switch back to Host
docker network inspect ct-bridge1
docker ps
docker attach ct-c2
ip addr
ping -c 5 ct-c1
Press Ctrl+P+Q, to switch back to Host


Task 2: Create a new docker bridge and check connectivity between containers of different bridges

docker network create --driver bridge ct-bridge2
docker run -it --network ct-bridge2 --name=ct-c3 busybox
Press Ctrl+P+Q, to switch back to Host
docker run -it --network ct-bridge2 --name=ct-c4 busybox 
Press Ctrl+P+Q, to switch back to Host
docker attach ct-c4
ping -c 5 ct-c3
ip addr
ping -c 5 ct-c1
ping -c 5 ct-c2
Press Ctrl+P+Q, to switch back to Host


Task 3: Using 'Docker network connect' command create a successful connection between containers of different bridges

docker network ls
docker network connect ct-bridge2 ct-c1
docker network inspect ct-bridge2
docker attach ct-c1
ping -c 5 ct-c4
ip addr
ip route

Task 4: Launch a container to host network

docker run -it --network host --name=ct-c5 busybox
ip addr
ifconfig
Press Ctrl+P+Q, to switch back to Host
docker network inspect host

Task 5: Launch a container to none network 

docker run -it --network none --name=ct-c6 busybox
ip addr
Press Ctrl+P+Q, to switch back to Host
