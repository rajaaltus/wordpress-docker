# Wordpress Deployment on Docker - Digital Ocean

Cloud Service: **Digital Ocean**
- Create Droplet(Otherwise create droplet with Docker image).
- Check the docker commands to ensure the installation.
 
 # Pulling necessary Docker images
- We need to pull images required to setup wordpress site. **Wordpress, MySQL, Phpmyadmin**

We using official images for wordpress and mysql. Installing all three images as separate containers.

## Wordpress
	$ docker pull wordpress:latest

used 'latest' tag to pull latest version. [Image link](https://hub.docker.com/_/wordpress)

## MySQL
	$ docker pull mysql:5.7

used 5.7 in this project.

## PhpMyAdmin
	$ docker pull phpmyadmin/phpmyadmin
Phpmyadmin to manage our database, tables and other database operations.

## Useful Docker basic commands
	$ docker login
	$ docker images
	$ docker ps -a 
	$ docker start <container_name>
	$ docker stop <container_name/id>

## Creating Docker-compose file (.yaml)

>version: '3'
  services:
	# Database
	db:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image: mysql:5.7
	volumes:
	&nbsp;&nbsp; - db_data:/var/lib/mysql
	restart: always
	environment:
	&nbsp;&nbsp;&nbsp;MYSQL_ROOT_PASSWORD: password
	&nbsp;&nbsp;&nbsp;MYSQL_DATABASE: wordpress
	&nbsp;&nbsp;&nbsp;MYSQL_USER: wordpress
	&nbsp;&nbsp;&nbsp;MYSQL_PASSWORD: wordpress
	networks:
		&nbsp;&nbsp; - wpsite
	# PhpMyAdmin
	phpmyadmin:
	&nbsp;&nbsp;&nbsp;depends_on:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - db
	image:
	&nbsp;&nbsp;&nbsp;phpmyadmin/phpmyadmin
	restart: always
	ports:
	&nbsp;&nbsp;&nbsp; - '8080:80'
	environment:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PMA_HOST: db
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MYSQL_ROOT_PASSWORD: password
	networks:
	&nbsp;&nbsp;&nbsp; - wpsite
	# Wordpress
	wordpress:
	&nbsp;&nbsp;&nbsp;depends_on:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- db
	image: 
	&nbsp;&nbsp;&nbsp;wordpress:latest
	ports:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - '8000:80'
	restart: always
	volumes: ['./:/var/www/html']
	environment:
	&nbsp;&nbsp;&nbsp;WORDPRESS_DB_HOST: db:3306
	&nbsp;&nbsp;&nbsp;WORDPRESS_DB_USER: wordpress
	&nbsp;&nbsp;&nbsp;WORDPRESS_DB_PASSWORD: wordpress
	&nbsp;&nbsp;&nbsp;WORDPRESS_DB_NAME: wordpress
networks:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - wpsite
links:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - db:mysql
networks:
&nbsp;&nbsp;&nbsp;wpsite:
volumes:
	db_data:


After Creating docker-compose file just run with the following command

	$ docker-compose up -d
-d flag help the container to run in detached mode.

Now you have all containers up and running for wordpress stack.

**Note**
Sometimes MySql will not allow remote ip to access the database. To fix that:

- Access your droplet with ssh
- Then access the running container mysql file using the following command
		
		 $ docker exec -it your_container_name_or_id bash

- Now you will be inside the active container. 
	
		$ mysql -u your_user -p
	now you will see a table like this:
```
+------------+------------------+
| host       | user             |
+------------+------------------+
| %          | root             |
| 127.0.0.1  | root             |
| ::1        | root             |
| localhost  | mysql.sys        |
| localhost  | root             |
| localhost  | sonar            |
+------------+------------------+
```

if you couldn't see **%** and **root** you will not be allowed to access from remote.

So, 
```
GRANT ALL PRIVILEGES 
 ON *.* TO 'remote'@'%' 
 IDENTIFIED BY 'safe_password' 
 WITH GRANT OPTION;`
```
edit with your **password** and **usename** and execute the above shown mysql query in the container bash. 

**Note:**
If you still persist Connection error. Check the mapped wordpress volume and find the **wp-config.php**. 


