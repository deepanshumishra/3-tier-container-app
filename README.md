**Deploying any 3-tier application**

We are going to deploy a 3-tier application using docker-compose
First task would be to choose application.
I’ll deploy NextCloud web application with MySQL as database and nginx as reverse proxy.

**1.	Creating docker-compose.yml**
```yaml
version: '3'
services:
  mysql:
    networks:
      - deepnetwork
    image: mysql:${MYSQL_VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 50M
    container_name: mysql
    restart: on-failure
    volumes:
      - "${DATA_DIR}/mysql:/var/lib/mysql"
    environment: 
      MYSQL_ROOT_PASSWORD: deepanshu

  nextcloud:
    networks:
      - deepnetwork
    image: nextcloud:${NEXTCLOUD_VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 50M
    container_name: nextcloud
    restart: on-failure
    volumes:
      - "${DATA_DIR}/nextcloud:/var/www/html"
      - "${DATA_DIR}/mysql:/var/lib/mysql"
    ports:
      - 8080:80
    links:
      - mysql:mysql
    environment:
      MYSQL_HOST: mysql
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: root
      MYSQL_PASSWORD: deepanshu
    depends_on:
      - mysql

  reverse-proxy:
    networks:
      - deepnetwork
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 50M
    container_name: nginx-proxy
    restart: on-failure
    ports:
      - 80:80
    depends_on:
      - nextcloud
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
networks:
  deepnetwork:
    driver: bridge
```
 
Now in the above file I’ve used few Variables to get them filled at build time we need to create a variable file called    .env in the present directory.
![image](https://user-images.githubusercontent.com/49090565/158077395-4eb9989c-c6b3-47e5-b80f-dca9e5b2c348.png)

 

I chose both versions of software which are compatible with each other.
The above docker-compose file will create persistent volume directory for all the data, and by setting restart attribute to on-failure we have instructed docker to restart the application container whenever they fail, and also in the same docker-compose file I am creating a network deepnetwork using bridge driver.

After this we need to create one nginx configuration file which will tell nginx container which we’ve setup as to work as reverse proxy, that what port to listen on and what service to show up on particular user request.

We’ll be creating a file named nginx.conf in same directory where docker-compose.yml is residing and content will be as follow:
![image](https://user-images.githubusercontent.com/49090565/158077411-c2739b60-3058-421d-8110-943b7688b177.png)


 

**2.	Running services using docker-compose**

Now, Run all the three services at once using a simple docker-compose command, as shown in below picture:
![image](https://user-images.githubusercontent.com/49090565/158077423-1f4c0fb3-cda8-4593-925f-7a7f1f84c70a.png)

 
Now, check the running applications using docker ps command, as shown below
![image](https://user-images.githubusercontent.com/49090565/158077429-f23c1e15-bd32-45f2-90a2-75e8e91212d6.png)

 
Head toward the localhost at 80 port  as per our nginx.conf configuration file we should land to NextCloud landing page, let’s see
![image](https://user-images.githubusercontent.com/49090565/158077442-5443b0b5-0490-41b6-a8e6-e139b6dbeb78.png)

 

Cool we are Done!!
