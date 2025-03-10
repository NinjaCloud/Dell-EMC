# Lab 6: Docker Networking

## Task 1: Create a new docker bridge and check connectivity between containers of the same bridge

1. List existing Docker networks:
   ```sh
   docker network ls
   ```
2. Create a new bridge network named `ct-bridge1`:
   ```sh
   docker network create --driver bridge ct-bridge1
   ```
3. Inspect the newly created network:
   ```sh
   docker network inspect ct-bridge1
   ```
4. Verify the network exists:
   ```sh
   docker network ls
   ```
5. Run a container `ct-c1` connected to `ct-bridge1`:
   ```sh
   docker run -it --network ct-bridge1 --name=ct-c1 busybox
   ```
6. Detach from the container using `Ctrl+P+Q`.
7. Run another container `ct-c2` in the same network:
   ```sh
   docker run -it --network ct-bridge1 --name=ct-c2 busybox
   ```
8. Detach from the container using `Ctrl+P+Q`.
9. Inspect the network again:
   ```sh
   docker network inspect ct-bridge1
   ```
10. List running containers:
    ```sh
    docker ps
    ```
11. Attach to `ct-c2`:
    ```sh
    docker attach ct-c2
    ```
12. Check the IP address:
    ```sh
    ip addr
    ```
13. Ping `ct-c1` to verify connectivity:
    ```sh
    ping -c 5 ct-c1
    ```
14. Detach using `Ctrl+P+Q`.

## Task 2: Create a new docker bridge and check connectivity between containers of different bridges

1. Create another bridge network `ct-bridge2`:
   ```sh
   docker network create --driver bridge ct-bridge2
   ```
2. Run a container `ct-c3` on `ct-bridge2`:
   ```sh
   docker run -it --network ct-bridge2 --name=ct-c3 busybox
   ```
3. Detach from the container using `Ctrl+P+Q`.
4. Run another container `ct-c4` on `ct-bridge2`:
   ```sh
   docker run -it --network ct-bridge2 --name=ct-c4 busybox
   ```
5. Detach from the container using `Ctrl+P+Q`.
6. Attach to `ct-c4`:
   ```sh
   docker attach ct-c4
   ```
7. Ping `ct-c3` to check connectivity within the same network:
   ```sh
   ping -c 5 ct-c3
   ```
8. Check the IP address:
   ```sh
   ip addr
   ```
9. Attempt to ping `ct-c1` and `ct-c2` (should fail since they are on `ct-bridge1`):
   ```sh
   ping -c 5 ct-c1
   ping -c 5 ct-c2
   ```
10. Detach using `Ctrl+P+Q`.

## Task 3: Using 'Docker network connect' command to create a successful connection between containers of different bridges

1. List all Docker networks:
   ```sh
   docker network ls
   ```
2. Connect `ct-c1` (from `ct-bridge1`) to `ct-bridge2`:
   ```sh
   docker network connect ct-bridge2 ct-c1
   ```
3. Inspect `ct-bridge2` to verify the connection:
   ```sh
   docker network inspect ct-bridge2
   ```
4. Attach to `ct-c1`:
   ```sh
   docker attach ct-c1
   ```
5. Ping `ct-c4` to test cross-network connectivity:
   ```sh
   ping -c 5 ct-c4
   ```
6. Check IP configuration and routing:
   ```sh
   ip addr
   ip route
   ```

## Task 4: Launch a container on the host network

1. Run a container `ct-c5` using the host network:
   ```sh
   docker run -it --network host --name=ct-c5 busybox
   ```
2. Check the IP address:
   ```sh
   ip addr
   ```
3. Use `ifconfig` to check the network configuration:
   ```sh
   ifconfig
   ```
4. Detach using `Ctrl+P+Q`.
5. Inspect the host network:
   ```sh
   docker network inspect host
   ```

## Task 5: Launch a container with no network

1. Run a container `ct-c6` with no network:
   ```sh
   docker run -it --network none --name=ct-c6 busybox
   ```
2. Check the IP address (should not have any network connectivity):
   ```sh
   ip addr
   ```
3. Detach using `Ctrl+P+Q`.
