# Docker Compose Project

Sample Docker Compose Project

The project contains four different components:
1. Nginx Server
2. Application Server
3. Database Server
4. NFS Server

The each component in the project has its own purpose. 

**1. Nginx Server**

The Nginx Server is used to distribute the load equally  among the Application Servers. The nginx_server container is configured to receive the traffic from external world. The nginx_server container is deployed in seperate network.

The `Environment` Variables that are required are as follows:

a. NGINX_PORT:
    The port on which the nginx_server container to be running, for example 80 or 8080.

 > **Note:**
 > SSL configuration is not yet completed in nginx_server docker image

b. APPLICATION_SERVER_1 and APPLICATION_SERVER_2
  The container name of the aplication_server hosting the application 



