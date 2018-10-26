### Usage

1) Load UCP client bundle


2) Create Docker Secret containing password for backup user

    ```
    echo "password" | docker secret create backuppass -
    ```

3) Schedule service
    ```
    docker service create -d \
      --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
      --mount source=dtrbackup,target=/tmp/backup \
      --restart-condition=none \
      --constraint=node.hostname==ip-172-31-17-16.ec2.internal \
      support/dtrbackup:latest
    ```

    ```
    compose example
    ```

