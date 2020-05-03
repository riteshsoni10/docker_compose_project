# Docker Compose Project
##### The Project Mainly Focuses on Infrastructure

<p align="center">
  <img src="/images/docker_compose_infra_final.jpg" width="650" title="Infrastructure Diagram">
  <br>
  <em>Fig.: Project Infrastructure Diagram </em>
</p>


### Pre-requisites:
1. OS: Redhat Enterprise  Linix 8 or Centos8
2. Docker Engine 
3. Docker-compose 


The project contains four different components:
1. Nginx Server
2. Application Server
3. Database Server
4. NFS Server

The each component in the project has its own purpose. 

#### Nginx Server

The Nginx Server is used to distribute the load equally  among the Application Servers. The nginx_server container is configured to receive the traffic from external world. The nginx_server container is deployed in seperate network.The container is launched with two network interfaces i.e.; application_external_network and application_internal_network.

Sample docker command
```
docker run -dit -p 80:80 -e NGINX_PORT=80 \
-e APPLICATION_SERVER=nodejs_application_server_1 \
-e APPLICATION_SERVER_2=nodejs_application_server_2 --name nginx_server \
--link nodejs_application_server_1 --link nodejs_application_server_2 \
riteshsoni296/nginx_server:latest
```

`--link` option is used for internal connectivity between the nginx container and application containers based on the container name

The `Environment` Variables that are required are as follows:

a. NGINX_PORT:
    The port on which the nginx_server container to be running, for example 80 or 8080.

 > **Note:**
 > SSL configuration is not yet completed in nginx_server docker image

b. APPLICATION_SERVER_1 and APPLICATION_SERVER_2
  The container name of the aplication_server container hosting the nodejs application 

The logs are stored in seperate volume named `logs_nginx` to preserve the logs even when the container is deleted or corrupted for debugging any issue in application. 


#### Application Server

The `NodeJS sample project` is deployed in the containers, it returns the names of persons stored in the mongodb Server. The nodejs_application_server_1 and nodejs_application_server_2 are attached with two network interfaces i.e the database network(database_internal_network) and application internal network(application_internal_network).

The containers depends on Mongo DB Database i.e mongo_db_server for GET API call to fetch details from the database and display the database entries. The code directory 


#### Database Server

MongoDB version 4.2.2 container image is used to launch the containers with some customisation fo the applications. The database server is launched  in seperate network to keep the data secure from the outside world.

The database network is secured from outside internet access by initialising the internal paramter in networks.

```
networks:
    database_internal_network:
        driver: bridge
        internal: "true"
        driver_opts:
            com.docker.network.enable_ipv6: "false"
        ipam:
            driver: default
            config:
                - subnet: "10.120.20.0/24"
```

The `Environment` variables that are passed i.e:

a. MONGO_INITDB_ROOT_USERNAME:

        It is used to define the root account user name in database server

b. MONGO_INITDB_ROOT_PASSWORD: 

        Password for the root account

c. MONGO_INITDB_USERNAME: 

        Application account user_name

d. MONGO_INITDB_PASSWORD: 

        Application User Account password

e. MONGO_INITDB_DATABASE: 

        Application Database
        
#### NFS Server

The NFS Server storage is used to share the /apps volume or application_code volume with application containers.

Docker Compose file to create a nfs volume

```
volumes:
    # Volume for Java Application Code
    application_code:
        driver: local
        driver_opts:
            type: "nfs4"
            o: "addr=10.150.20.12,rw"
            device: ":/apps"
```

Here,
    `addr` is NFS Server IP. It can be NFS Server IP or NFS Server DNS name.
    `device` is configured as /apps instead of /nfsshare/apps, in NFS version 4 due to setting of `fsuid=0` in exports file in NFS Server, the shared volume /nfsshare can be mounted as /.
    
HealthCheck is configured to enable of NFS Server container to start and ready first before application containers.

```
    healthcheck:
            test: ["CMD", "netstat", "-tnlp", "|grep", "2049"]
            interval: 60s
            timeout: 10s
            retries: 5
```
Here,
    `test` paramter is configured to check the availability of the NFS server till `interval` seconds. 
    If the request does not receives any response between `timeout` seconds, the service is marked as unhealthy.
    
The NFS Server is executed as root user to run some priviledged commands i.e mount and writinf mounts in fstab file. It is achieved in docker compose by initialising the priviledged flag as true

```
        privileged: "true"
```
    
The `Environment` variables that are to be passed i.e:

a. SHARED_DIRECTORY and SHARED_DIRECTORY_2

The dicrectories that are mounted in application containers to store the application code in one volume rather than having multiple copies of the same code in different containers.

b. SYNC

The environment variable sets the Mount Option type as sync or async. If the Environment variable is nt defined, the container sets the default mount option to aync.  

The option `sync` means that all changes to the according filesystem are immediately flushed to disk; the respective write operations are being waited for. In contrast, with `async` the system buffers the write operation and optimizes the actual writes; meanwhile, instead of being blocked the process in userland continues to run.


