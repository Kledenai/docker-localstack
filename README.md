# LocalStack

A LocalStack and MinIO service to run on docker for local AWS development

## running the application without docker compose

We will be using `docker run` to be able to run **getting-started**, before running the service we will need create the volume and network to be used for localstack, this is only in the case for you use the `docker run` in the `docker compose` you won't need create a volume and network before run a service, follow the command below to create a volume:

```bash
docker volume create localstack
```

After running the volume creation command, run the command below to view the created volume:

```bash
docker volume ls
```

This is the expected return:

```bash
DRIVER    VOLUME NAME
local     localstack
```

Having successfully created the local volume, follow the command bellow to create a network:

```bash
docker network create localstack --attachable --driver bridge
```

Note that I am using two options in creating the network, below are the options and what each of them do:

- **attachable**: Enable manual container attachment
- **driver**: Driver to manage the Network

After running the network creation command, run the command below to view the created network:

```bash
docker network ls
```

This is the expected return:

```bash
NETWORK ID     NAME        DRIVER    SCOPE
4eba0f206c3a   localstack  bridge    local
```

Everything working out we can go to the next step, follow the line below to able to run the localstack service:

```bash
docker run -dp 4566:4566 -e SERVICES="s3,sqs,sns" -e DEBUG=1 -e AWS_ACCESS_KEY_ID="test" -e AWS_SECRET_ACCESS_KEY="test" -e AWS_DEFAULT_REGION="us-east-1" --volume localstack:/tmp/localstack --network localstack localstack/localstack:latest
```

Everything working out we can go to the next step, follow the line below to able to run the minio service:

```bash
docker run -dp 9000:9000 -dp 9001:9001 -e MINIO_ROOT_USER="minioadmin" -e MINIO_ROOT_PASSWORD="minioadmin" --volume ./minio/data:/data --network localstack quay.io/minio/minio server /data --console-address ":9001"
```

Note that we are using the same network as **localstack**, as we need to make the containers have this entry so that **minio** can access **localstack**.

After the services running you can run the command below to check the health of the containers:

```bash
docker ps
```

This is the expected return:

```bash
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                        NAMES
dc750a9d1e39   quay.io/minio/minio         "/usr/bin/docker-ent…"   22 seconds ago   Up 16 seconds   0.0.0.0:9000->9000/tcp, 0.0.0.0:9001->9001/tcp   minio
647bf9d5186e   localstack/localstack:latest "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    0.0.0.0:4566->4566/tcp, :::4566->4566/tcp        localstack
```

Having had this return with this result, let's access the services, in your browser access `localhost:9001` and you will see the **minio** console login screen.

## running the application with docker compose

We will use docker-compose to be able to run the script already prepared to be able to run the service in docker, follow the example below of what is expected to be in the file:

```bash
version: "3.9"

services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "${LOCALSTACK_PORT}:4566"
      - "4571:4571"
    environment:
      - SERVICES=${LOCALSTACK_SERVICES}
      - DEBUG=${LOCALSTACK_DEBUG}
      - DATA_DIR=/tmp/localstack/data
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
      - EDGE_PORT=4566
      - LOCALSTACK_HOST=localhost
    volumes:
      - "./localstack:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
    networks:
      - localstack

  minio:
    image: quay.io/minio/minio
    container_name: minio
    command: server /data --console-address ":9001"
    ports:
      - "${MINIO_API_PORT}:9000"
      - "${MINIO_CONSOLE_PORT}:9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - ./minio/data:/data
    networks:
      - localstack

  minio-client:
    image: minio/mc
    container_name: minio-client
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
        sleep 5;
        mc alias set local http://minio:9000 ${MINIO_ROOT_USER} ${MINIO_ROOT_PASSWORD};
        mc mb -p local/${MINIO_BUCKET_NAME};
        mc admin info local;
        tail -f /dev/null
      "
    networks:
      - localstack

volumes:
  localstack:
  minio:

networks:
  localstack:
    driver: bridge
```

Note that variable references are being passed, which are being taken from a .env file, you will need to have an .env file in your project root containing the same parameters below:

```bash
LOCALSTACK_PORT=
LOCALSTACK_SERVICES=
LOCALSTACK_DEBUG=

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=

MINIO_API_PORT=
MINIO_CONSOLE_PORT=
MINIO_ROOT_USER=
MINIO_ROOT_PASSWORD=
MINIO_BUCKET_NAME=
```

Add the values ​​and then save the file, having done that we can run the file.

Only this is enough for us to run the service, if everything is ok, run the command below:

```bash
docker-compose up -d
```

After run the command you can run the command below to check the health of the containers:

```bash
docker ps
```

This is the expected return:

```bash
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                        NAMES
dc750a9d1e39   quay.io/minio/minio         "/usr/bin/docker-ent…"   22 seconds ago   Up 16 seconds   0.0.0.0:9000->9000/tcp, 0.0.0.0:9001->9001/tcp   minio
647bf9d5186e   localstack/localstack:latest "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    0.0.0.0:4566->4566/tcp, :::4566->4566/tcp        localstack
```

Having had this return with this result, let's access the services, in your browser access `localhost:9001` and you will see the **minio** console login screen.

The service name has been set to `localstack`, this will be the **hostname** used when you try to access the server via **localstack**.

Everything being in agreement you have already managed to run the localstack in docker.
