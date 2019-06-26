# Setup InfluxDB and Grafana running in Docker containers to store and graph sensor data 
### Author: [Olav Tollefsen](https://www.linkedin.com/in/olavtollefsen/)

## Introduction

This article documents the process of installing InfluxDB and Grafana running in Docker containers.

### Prerequsites

- A computer running as a Docker host

## Create a user-defined bridge network
```
 $ docker network create influxdb_grafana_network
```

## Run InfluxDB in a Docker container

To run InfluxDB in a Docker container, first create a Dcocker Volume to permanently store the InfluxDB data files.

### Create a Docker Volume for InfluxDB

```
 $ docker volume create influxdb_data
```

### Run InfluxDB in a Docker container

```
    $ docker run --name="influxdb" \
        -p 8086:8086 \
        -v influxdb_data:/var/lib/influxdb \
        --network influxdb_grafana_network \
        --restart on-failure \
        influxdb
```

Alternative to bind permanent storage in the container to a directory on the host:

```
    $ docker run --name="influxdb" \
        -p 8086:8086 \
        --mount type=bind,source=/mnt/influxdb,target=/var/lib/influxdb \
        --network influxdb_grafana_network \
        --restart on-failure \
        influxdb
```

### Check the total size of the Docker Volume for InfluxDB

```
$ du --summarize --human-readable /var/snap/docker/common/var-lib-docker/volumes/influxdb_data/_data
```

### Display detailed information about the Docker Volume for InfluxDB

```
 $ docker volume inspect influxdb_data
```

### Make an archive of the InfluxDB data files (remember to stop the InfluxDB container first)

```
$ sudo tar -cvpf influxdb.tar -C <path from the Mountpoint section of the docker inspect volume command> .
```

### Restore an archive of the InfluxDB data files (remember to stop the InfluxDB container first)

```
$ sudo tar --same-owner -xvf influxdb.tar -C <path from the Mountpoint section of the docker inspect volume command>
```

## Run Grafana in a Docker container

### Create a Docker Volume for Grafana

```
 $ docker volume create grafana_data
```

### Run Grafana in a Docker container

```
$ docker run --name grafana \
  -p 3000:3000 \
  -v grafana_data:/var/lib/grafana \
  --network influxdb_grafana_network \
  --restart on-failure \
  grafana/grafana
```

Alternative to bind permanent storage in the container to a directory on the host:

```
$ docker run --name grafana \
  -p 3000:3000 \
  --mount type=bind,source=/mnt/grafana,target=/var/lib/grafana \
  --network influxdb_grafana_network \
  grafana/grafana
```

After the container launched successfully, you can open the Grafana web page by http://<Ip address of Docker host>:3000. Login with default admin/admin user, find and click on the the Add data dources button in the upper-left section of the user interface and then click on InfluxDB.

Use http://influxdb:8086 as the Url. This works, since both the InfluxDB container and the Grafana container is connnected to the same user-defined bridge network.

### Display detailed information about the Docker Volume for Grafana

```
 $ docker volume inspect grafana_data
```

### Make an archive of the Grafana data files (remember to stop the Grafana container first)

```
$ sudo tar -cvpf grafana.tar -C <path from the Mountpoint section of the docker inspect volume command> .
```

### Restore an archive of the Grafana data files (remember to stop the Grafana container first)

```
$ sudo tar --same-owner -xvf grafana.tar -C <path from the Mountpoint section of the docker inspect volume command> 
```
