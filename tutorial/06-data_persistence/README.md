#[Data Persistence] Redis

##Overview

https://redis.io/

***Redis*** is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries.

***Redis*** Network Connectivity:  ***PORT 6379***

There are a couple of things in the Redis specification that make data persist between deployments of this stack:

- ***Redis*** always runs on the manager, so it’s always using the same filesystem.
- ***Redis*** accesses an arbitrary directory in the host’s file system as ***~/data*** inside the container, which is where ***Redis*** stores data.

Together, this is creating a “source of truth” in your host’s physical filesystem for the ***Redis*** data. Without this, ***Redis*** would store its data in ***~/data*** inside the container’s filesystem, which would get wiped out if that container were ever redeployed.

This source of truth has two components:

- The placement constraint you put on the ***Redis*** service, ensuring that it always uses the same host.
- The volume you created that lets the container access ***~/data*** (on the host) as ***/data*** (inside the ***Redis*** container). While containers come and go, the files stored on ***~/data*** on the specified host persists, enabling continuity.

From <https://docs.docker.com/get-started/part5/#persist-the-data> 

**NOTE:**

While we are limiting the file system to a single Container to match the SWARM MASTER, there are other methods to extend persistent storage to replicas of services that need this. GlusterFS could be used perhaps.

###Updated docker-compose.yml

    version: "3"
    services:
      web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
    condition: on-failure
      resources:
    limits:
      cpus: "0.1"
      memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
      visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
    constraints: [node.role == manager]
    networks:
      - webnet
      redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
    constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
    networks:
      webnet:

From <https://docs.docker.com/get-started/part5/#persist-the-data> 

###Create a ***~/data*** directory on the SWARM MANAGER


    vCon01:~$ mkdir ~/data

This maps to the ***services::redis::volumes:*** value in the ***docker-compose.yml*** above - needs to match your working directory and user. In my case, this is ***/home/jc3066/data***

###Update Stack and Re-Deploy (from SWARM MASTER)

    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ docker stack deploy -c docker-compose.yml 03-services
    Creating service 03-services_redis
    Updating service 03-services_web (id: kiqju7nlhmhbjrtkkve7fyg7b)
    Updating service 03-services_visualizer (id: m4k5d8r5dvzfu5flein6cwr5e)
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ docker service ls
    ID  NAME MODEREPLICASIMAGE   PORTS
    otlqlqs22i6x03-services_redisreplicated  1/1 redis:latest*:6379->6379/tcp
    jrbc2coedqzm03-services_visualizer   replicated  1/1 dockersamples/visualizer:stable *:8080->8080/tcp
    ialaouyk7c9b03-services_web  replicated  5/5 f0otsh0t/tutorial:02-containers.friendlyhello   *:80->80/tcp
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ docker stack ps 03-services
    ID  NAME   IMAGE   NODEDESIRED STATE   CURRENT STATE   ERROR   PORTS
    vtorhlabhlu503-services_redis.1redis:latestvCon01  Running Running 1 second ago
    ydzgxa8mjdd903-services_visualizer.1   dockersamples/visualizer:stable vCon01  Running Running 3 seconds ago   
    npzf2w5sxu6d03-services_web.1  f0otsh0t/tutorial:02-containers.friendlyhello   vCon02  Running Running 5 seconds ago   
    txtgrro8v3ff03-services_web.2  f0otsh0t/tutorial:02-containers.friendlyhello   vCon03  Running Running 4 seconds ago   
    oncub0xhsukl03-services_web.3  f0otsh0t/tutorial:02-containers.friendlyhello   vCon01  Running Running 6 seconds ago   
    zostyjn2vwst03-services_web.4  f0otsh0t/tutorial:02-containers.friendlyhello   vCon02  Running Running 4 seconds ago   
    ixjmt5jf5so003-services_web.5  f0otsh0t/tutorial:02-containers.friendlyhello   vCon03  Running Running 4 seconds ago   
    
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ docker ps
    CONTAINER IDIMAGE   COMMAND  CREATED STATUS  PORTS   NAMES
    fb51a3b56a95redis:latest"docker-entrypoint.sâ€¦"   4 minutes ago   Up 4 minutes6379/tcp03-services_redis.1.vtorhlabhlu5fnvg8gls3hzkm
    482ac52541b0dockersamples/visualizer:stable "npm start"  4 minutes ago   Up 4 minutes8080/tcp03-services_visualizer.1.ydzgxa8mjdd96pj5ihau7x9x1
    0ec6ddaaeae5f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py"  4 minutes ago   Up 4 minutes80/tcp  03-services_web.3.oncub0xhsuklup9gyl2qk1hr0

###Verify Application Services

    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:8080
    <!doctype html>
    <html>
    <head>
      <meta charset="utf-8">
      <title>Visualizer</title>
      <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
      <meta name="description" content="">
      <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
      <link href='//fonts.googleapis.com/css?family=Ubuntu+Mono|Open+Sans:400,700,400italic' rel='stylesheet' type='text/css'>
      <style type="text/css">
       .hidden{ display: none; }
      </style>
    </head>
    <body style='background:#254356'>
      <div class='tabs'>
    <button id='tab-physical'>
      <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 80 80"><path fill="#FFF" d="M14.752 32.456l-7.72.002v7.553h7.72v-7.554zm9.65 0h-7.72v7.556h7.72v-7.556zm0-9.445h-7.72v7.556h7.72V23.01zm9.65 9.446h-7.72v7.556h7.72v-7.556zm0-9.445h-7.72v7.556h7.72V23.01zm9.648 9.446h-7.72v7.556h7.72v-7.556zm0-9.445h-7.72v7.556h7.72V23.01zm9.65 9.446l-7.72.002v7.553h7.72v-7.554zm-9.65-18.89h-7.72v7.556h7.72v-7.556zm31.938 23.106c-2.51-1.417-5.85-1.61-8.693-.792-.35-2.958-2.337-5.55-4.7-7.41l-.938-.738-.79.89c-1.58 1.79-2.052 4.768-1.838 7.053.16 1.68.697 3.388 1.756 4.737-.805.473-1.717.85-2.53 1.12-1.657.55-3.456.854-5.206.854H3.544l-.105 1.107c-.354 3.7.165 7.402 1.728 10.778l.673 1.343.078.124c4.622 7.68 12.74 10.914 21.584 10.914 17.125 0 31.248-7.48 37.734-23.284 4.335.222 8.77-1.033 10.89-5.082l.54-1.033-1.028-.578zm-57.77 19.982v.002c-2.18 0-3.955-1.735-3.955-3.866 0-2.132 1.774-3.866 3.954-3.866s3.954 1.732 3.954 3.865c0 2.13-1.77 3.864-3.95 3.864zm-.01-5.854c-1.137 0-2.06.9-2.06 2.013 0 1.11.924 2.01 2.06 2.01 1.134 0 2.057-.9 2.057-2.01 0-1.11-.922-2.013-2.057-2.013z"/></svg>
    </button>
    
      </div>
      <div id="app">
    <!-- content goes here -->
      </div>
    
      <script type="text/javascript">
    window.MS = '1000';
      </script>
      <script type="text/javascript" src="app.js"></script>
    </body>
    </html>
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:80
    <h3>Hello World!</h3><b>Hostname:</b> 23d3eaf18ea7<br/><b>Visits:</b> 1
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:80
    <h3>Hello World!</h3><b>Hostname:</b> ee39aee4c950<br/><b>Visits:</b> 2
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:80
    <h3>Hello World!</h3><b>Hostname:</b> fa4834e0404a<br/><b>Visits:</b> 3
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:80
    <h3>Hello World!</h3><b>Hostname:</b> 009250d1423f<br/><b>Visits:</b> 4
    
    vCon01:~/gitw/yoyodyne/poc-docker/tutorial/06-data_persistence$ curl -4 http://localhost:80
    <h3>Hello World!</h3><b>Hostname:</b> 0ec6ddaaeae5<br/><b>Visits:</b> 5

^^ Note Visits now are being recorded as data is now stored in Redis Service. This is returned by the ***app.py*** from the 02-Containers Module

    vCon01:~/gitw/yoyodyne/poc-docker$ docker ps
    CONTAINER IDIMAGE   COMMAND  CREATED STATUS  PORTS   NAMES
    fb51a3b56a95redis:latest"docker-entrypoint.sâ€¦"   39 minutes ago  Up 39 minutes   6379/tcp03-services_redis.1.vtorhlabhlu5fnvg8gls3hzkm
    482ac52541b0dockersamples/visualizer:stable "npm start"  39 minutes ago  Up 39 minutes   8080/tcp03-services_visualizer.1.ydzgxa8mjdd96pj5ihau7x9x1
    0ec6ddaaeae5f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py"  39 minutes ago  Up 39 minutes   80/tcp  03-services_web.3.oncub0xhsuklup9gyl2qk1hr0
    
    vCon01:~/gitw/yoyodyne/poc-docker$ docker exec -i -t fb51a3b56a95 bash

^^ Executing BASH from Redis Container

    root@fb51a3b56a95:/data# which redis-cli
    /usr/local/bin/redis-cli
    
    root@fb51a3b56a95:/data# redis-cli -h 127.0.0.1 -p 6379 --stat
    ------- data ------ --------------------- load -------------------- - child -
    keys   mem  clients blocked requestsconnections  
    1  910.00K  6   0   27 (+0) 23  
    1  910.00K  6   0   28 (+1) 23  
    1  910.00K  6   0   29 (+1) 23  
    1  910.00K  6   0   30 (+1) 23  
    1  910.00K  6   0   31 (+1) 23  
    1  910.00K  6   0   32 (+1) 23  
    1  910.00K  6   0   33 (+1) 23  
    1  910.00K  6   0   34 (+1) 23  
    1  910.00K  6   0   35 (+1) 23  
    1  910.00K  6   0   36 (+1) 23  
    1  910.00K  6   0   37 (+1) 23  
    1  910.00K  6   0   38 (+1) 23  
    1  910.00K  6   0   39 (+1) 23  
    1  910.00K  6   0   40 (+1) 23  
    1  910.00K  6   0   41 (+1) 23  
    1  910.00K  6   0   42 (+1) 23  
    1  910.00K  6   0   43 (+1) 23  
    1  910.00K  6   0   44 (+1) 23  
    1  910.00K  6   0   45 (+1) 23  
    1  910.00K  6   0   46 (+1) 23  
    
    root@fb51a3b56a95:/data# redis-cli -h 127.0.0.1 -p 6379 --scan
    Counter

^^ Redis list of keys shows only one of "Counter" 

    root@fb51a3b56a95:/data# redis-cli -h 127.0.0.1 -p 6379 monitor
    OK
    1519257297.916979 [0 10.0.0.10:39324] "INCRBY" "counter" "1"

^^ We see Redis increment the number of Visits by "1" every time I refresh the browser for the Web Page @ port 80.


## App Operations ##

**Scale the Application:**

Modify the replicas value in the ***docker-compose.yml*** to scale down or up the number of containers that run the service in the application.  Then rerun the docker stack deploy command:

    $ docker stack deploy -c docker-compose.yml 03-services


**Take down the Application and Swarm:**

    $ docker stack rm 03-services
    $ docker swarm leave --force



**Command Reference**

    docker stack ls# List stacks or apps
    docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
    docker node ls # List running containers in SWARM
    docker service ls # List running services associated with an app
    docker service ps <service>  # List tasks associated with an app
    docker inspect <task or container>   # Inspect task or container
    docker container ls -q  # List container IDs
    docker stack rm <appname> # Tear down an application
    docker swarm leave --force  # Take down a single node swarm from the manager


###Terminal Recording

https://asciinema.org/a/113840
