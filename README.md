# STEP 1 - INIT DOCKER COMPOSE
>>> command: docker-compose up -d


# STEP 2 - SETUP NGINX LOAD BALANCER CONTAINER

Install bash and vim to container
 >>> command: docker-compose exec nginx apk add bash vim

Access container using bash
>>> command: docker-compose exec nginx bash

# STEP 3 - SET NGINX.CONF FILE

Edit NGINX.CONF File
>>> command: vim /etc/nginx/nginx.conf
Change '$remote_addr' to '$http_x_real_ip' to capture the user IP instead of the node IP


# STEP 4 - SET DEFAULT.CONF FILE LOAD BALANCER
Edit DEFAULT.CONF File -> Where we set the different container environments on Load Balancer
>>> command: vim /etc/nginx/conf.d/default.conf
Add the 'upstream' variable with the nodes at the top of the file:

```
upstream nodes{
    server node1;
    server node2;
    server node3;
}
```

On 'location' set the Reverse proxy using:

```
    proxy_pass http://nodes;
    proxy_set_header X-Real-IP $remote_addr;
```

USE Command to test and see if will run
>>> command: nginx -t

USE Command to reload the webserver
>>> command: nginx -s reload


# STEP 5 - SETUP ACCESS LOGS
Edit DEFAULT.CONF File -> Where we set the different container environments on Load Balancer
>>> command: vim /etc/nginx/conf.d/default.conf
Remove the comment from the access_log and rename it to 'nginx-access.log'; remove 'main;' at the end (nginx or node1 or node2 or node3);

```
    access_log /var/log/nginx/nginx-access.log; (nginx-access or node1-access or node2-access or node3-access);
``` 

USE Command to test and see if will run
>>> command: nginx -t

USE Command to reload the webserver
>>> command: nginx -s reload

# STEP 6 - SETUP NODES 1,2 and 3

Add the bash and vim to each node using:
>>> command: docker-compose exec node1 apk add bash vim

Run bash on each node
>>> command: docker-compose exec node1 bash

Edit index.html to differentiate between nodes
>>> command: vim /usr/share/nginx/html/index.html

Repeat STEP 5 to each node to add the access log. And then repeat the STEP 6 for all nodes.