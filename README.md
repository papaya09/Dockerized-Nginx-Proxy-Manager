# Secure Setup of Nginx Proxy Manager with Docker

This guide provides detailed steps on how to deploy and secure Nginx Proxy Manager using Docker and Portainer. It covers everything from the initial Docker setup to securing access to the Nginx Proxy Manager interface.

## Prerequisites
- Docker installed on your machine.
- Basic understanding of Docker commands and Portainer.

## Installation and Configuration Steps

### 1. Create a Docker Volume for Portainer Data
```
docker volume create portainer_data
```

###2. Run Portainer
```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

### 3. Configure Nginx Proxy Manager in Portainer
- Access Portainer at http://yourip:9000.
- Go to "Stacks" and create a new stack called "nginxproxymanager".
- Use the following configuration in the web editor:
```
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "youruser"
      DB_MYSQL_PASSWORD: "yourpassword"
      DB_MYSQL_NAME: "npm"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
  db:
    image: 'jc21/mariadb-aria:latest'
    environment:
      MYSQL_ROOT_PASSWORD: "yourpasswordforroot"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "youruser"
      MYSQL_PASSWORD: "yourpassword"
    volumes:
      - ./mysql:/var/lib/mysql

```

### 4. Access and Configure Proxy Hosts
- Access the Nginx Proxy Manager at http://yourip:81.
- Login with admin@example.com / changeme.
- Configure Proxy Hosts as follows:
  - For Portainer:
    - Domain Name: yoursubdomain.domain
    - Scheme: http
    - Forward Hostname/IP: localhost
    - Forward Port: 9000
    - Enable SSL, HTTP/2, and other security settings as needed.
  - For Nginx Proxy Manager Dashboard:
    - Domain Name: yoursubdomain.domain
    - Scheme: http
    - Forward Hostname/IP: localhost
    - Forward Port: 81
    - Enable SSL, HTTP/2, and other security settings as needed.

### 5. Restart Portainer in the Correct Network
```
docker stop portainer
```
```
docker rm portainer
```
```
docker run -d -p 8000:8000 --network nginxproxymanager_default --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/port
```

### 6. Secure Nginx Proxy Manager Access
- To restrict access to the Nginx Proxy Manager dashboard to only your desired domain, follow these steps:
- Get the container ID of your Nginx Proxy Manager instance using:
```
docker ps
```
- Access the container's bash shell:
```
docker exec -it [container_id] /bin/bash
```
- Navigate to the Nginx configuration directory:
```
cd /etc/nginx/conf.d/
```
- Copy the default configuration file:
```
docker cp [container_id]:/etc/nginx/conf.d/default.conf ./default.conf
```
- Edit the default configuration file:
```
nano default.conf
```
- Add the following server block to restrict access by IP:
```
server {
    listen 81;
    server_name yourserverip;  # Only respond to this server IP

    location / {
        if ($host != 'proxy.patroldex.com') {
            return 403;  # Deny access if not accessed via the correct domain
        }
        proxy_pass http://127.0.0.1:81;  # Ensure this points to where the Nginx Proxy Manager dashboard is actually running
    }
}

```
- Copy the modified configuration file back to the container:
```
docker cp ./default.conf [container_id]:/etc/nginx/conf.d/default.conf
```
- Test the Nginx configuration:
```
nginx -t
```
- Reload Nginx to apply changes:
```
nginx -s reload
```
