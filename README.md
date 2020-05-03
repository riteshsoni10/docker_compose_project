# Docker Compose Project

Sample Docker Compose Project

The project contains four different components:
1. Nginx Server
2. Application Server
3. Database Server
4. NFS Server

The each component in the project has its own purpose. 
1. Nginx Server

The Nginx Server is used to distribute the load equally  among the Application Servers. The nginx_server container is configured to receive the traffic from external world. The nginx_server container is deployed in seperate container.
