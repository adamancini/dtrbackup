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
      --restart-delay=24h \
      support/dtrbackup:2.7
    ```

```
version: '3.7'
services:
  dtr:
    deploy:
      restart_policy:
        condition: any
        delay: 24h
      placement:
        constraints:
          - node.role == worker
          - node.labels.com.docker.ucp.collection.system == true
    image: adamancini/dtrbackup:2.7
    environment:
      UCP_USER: admin
      UCP_URL: ucp.test.mira.annarchy.net
    secrets:
      - source: backuppass
        target: password
    volumes:
      - source: dtrbackup
        target: /backup
        type: volume
      - source: /var/run/docker.sock
        target: /var/run/docker.sock
        type: bind
  ucp:
    deploy:
      restart_policy:
        condition: any
        delay: 24h
      placement:
        constraints:
          - node.role == manager
    image: adamancini/ucpbackup:3.2
    environment:
      UCP_USER: admin
    secrets:
      - source: backuppass
        target: password
    volumes:
      - source: ucpbackup
        target: /backup
        type: volume
      - source: /var/run/docker.sock
        target: /var/run/docker.sock
        type: bind
volumes:
  ucpbackup:
  dtrbackup:
secrets:
  backuppass:
    external: true
```
