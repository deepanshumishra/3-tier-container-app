**Deploying any 3-tier application**

We are going to deploy a 3-tier application using docker-compose
First task would be to choose application.
I’ll deploy NextCloud web application with MySQL as database and nginx as reverse proxy
**1.	Creating docker-compose.yml**

version: '3'
services:
  mysql:
    networks:
      - deepnetwork
    image: mysql:${MYSQL_VERSION}
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
 
Now in the above file I’ve used few Variables to get them filled at build time we need to create a variable file called    .env in the present directory.
 

I chose both versions of software which are compatible with each other.
The above docker-compose file will create persistent volume directory for all the data, and by setting restart attribute to on-failure we have instructed docker to restart the application container whenever they fail, and also in the same docker-compose file I am creating a network deepnetwork using bridge driver.

After this we need to create one nginx configuration file which will tell nginx container which we’ve setup as to work as reverse proxy, that what port to listen on and what service to show up on particular user request.

We’ll be creating a file named nginx.conf in same directory where docker-compose.yml is residing and content will be as follow:

 

**2.	Running services using docker-compose**
Now, Run all the three services at once using a simple docker-compose command, as shown in below picture:
 
Now, check the running applications using docker ps command, as shown below
 
Head toward the localhost at 80 port  as per our nginx.conf configuration file we should land to NextCloud landing page, let’s see
 

Cool we are Done!
