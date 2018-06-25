#SWARM Overview:

Minimum of 3 Nodes to form a Swarm: Swarm Manager & Worker Nodes. This will utilize the service containers created in the previous 03-services module.

##IP Ports:

Ports used for Docker Swarm - The following ports must be available. On some systems, these ports are open by default.

- TCP port 2377 for cluster management communications
- TCP and UDP port 7946 for communication among nodes
- UDP port 4789 for overlay network traffic

If you plan on creating an overlay network with encryption (--opt encrypted), you also need to ensure ip protocol 50 (ESP) traffic is allowed.

##SWARM INIT

###vCon01 (@ 10.0.2.111) - SWARM MANAGER

    vCon01:~$ docker swarm init --advertise-addr 10.0.2.111
    Swarm initialized: current node (z6uf2kq0luy8l81z95qqlw9qk) is now a manager.
    
    To add a worker to this swarm, run the following command:
    
    docker swarm join --token SWMTKN-1-4twm43u4vv4msiljw9adcvn7kidggbj9uw449c9e372ec9rqgf-ecbctam2exwd37exvphrbb0vg 10.0.2.111:2377
    
    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

###vCon02 (@ 10.0.2.112) - WORKER NODE

    vCon02:~$ docker swarm join --token SWMTKN-1-4twm43u4vv4msiljw9adcvn7kidggbj9uw449c9e372ec9rqgf-ecbctam2exwd37exvphrbb0vg 10.0.2.111:2377
    This node joined a swarm as a worker.

###vCon03 (@ 10.0.2.113) - WORKER NODE

    vCon03:~$ docker swarm join --token SWMTKN-1-4twm43u4vv4msiljw9adcvn7kidggbj9uw449c9e372ec9rqgf-ecbctam2exwd37exvphrbb0vg 10.0.2.111:2377
    This node joined a swarm as a worker.

###vCon01 (@ 10.0.2.111) - SWARM MANAGER

    vCon01:~$ docker node ls
    IDHOSTNAMESTATUS  AVAILABILITYMANAGER STATUS
    z6uf2kq0luy8l81z95qqlw9qk *   vCon01  Ready   Active  Leader
    rk382d2dtb8w7rsw5p8kyur0o vCon02  Ready   Active  
    28riqpdndxz4xz3xfnsn57ojs vCon03  Ready   Active


##START STACK

### vCon01 (@ 10.0.2.111) - SWARM MANAGER with 'replicas: 2'

    vCon01:~/docker/03-services$ docker stack deploy -c docker-compose.yml 03-services
    Creating network 03-services_webnet
    Creating service 03-services_web
    
    vCon01:~/docker/03-services$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    95983640e20df0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 36 seconds ago  Up 35 seconds   80/tcp  03-services_web.2.3g5wvfko4to2e7o186aclqgvg
    jc3066@vCon01:~/docker/03-services$ docker stack ps 03-services
    
    vCon01:~/docker/03-services$ docker stack ps 03-services
    ID  NAMEIMAGE   NODEDESIRED STATE   CURRENT STATEERROR   PORTS
    vedbrol0pizt03-services_web.1   f0otsh0t/tutorial:02-containers.friendlyhello   vCon02  Running Running about a minute ago   
    3g5wvfko4to203-services_web.2   f0otsh0t/tutorial:02-containers.friendlyhello   vCon01  Running Running 3 minutes ago   

###vCon02 (@ 10.0.2.112) - WORKER NODE

    ## vCon02 needs to download images from Docker Hub
    vCon02:~$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED  STATUS  PORTS   NAMES
    ce1cbc1b49ecf0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" About a minute ago   Up About a minute   80/tcp  03-services_web.1.vedbrol0piztelokq42yby1fa


##MODIFY STACK

###vCon01 (@ 10.0.2.111) - SWARM MANAGER

    ## MODIFY docker-compose.yml 'replicas: 2' => 'replicas: 5'
    
    vCon01:~/docker/03-services$ docker stack deploy -c docker-compose.yml 03-services
    Updating service 03-services_web (id: kiqju7nlhmhbjrtkkve7fyg7b)
    
    vCon01:~/docker/03-services$ docker stack ps 03-services
    ID  NAMEIMAGE   NODEDESIRED STATE   CURRENT STATE  ERROR   PORTS
    vedbrol0pizt03-services_web.1   f0otsh0t/tutorial:02-containers.friendlyhello   vCon02  Running Running 6 minutes ago  
    3g5wvfko4to203-services_web.2   f0otsh0t/tutorial:02-containers.friendlyhello   vCon01  Running Running 8 minutes ago  
    ogxzbvhaf79o03-services_web.3   f0otsh0t/tutorial:02-containers.friendlyhello   vCon03  Running Preparing 23 seconds ago   
    t123lbxp6vq903-services_web.4   f0otsh0t/tutorial:02-containers.friendlyhello   vCon02  Running Running 22 seconds ago 
    j3xhf9f3eylr03-services_web.5   f0otsh0t/tutorial:02-containers.friendlyhello   vCon03  Running Preparing 23 seconds ago  
    
    vCon01:~/docker/03-services$ docker service ls
    ID  NAMEMODEREPLICASIMAGE   PORTS
    kiqju7nlhmhb03-services_web replicated  5/5 f0otsh0t/tutorial:02-containers.friendlyhello   *:80->80/tcp
    
    vCon01:~/docker/03-services$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    95983640e20df0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 20 minutes ago  Up 20 minutes   80/tcp  03-services_web.2.3g5wvfko4to2e7o186aclqgvg

###vCon02 (@ 10.0.2.112) - WORKER NODE

    vCon02:~$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    0f2af86b6b1ef0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 58 seconds ago  Up 57 seconds   80/tcp  03-services_web.4.t123lbxp6vq99y9wcqfho7l9h
    ce1cbc1b49ecf0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 7 minutes ago   Up 7 minutes80/tcp  03-services_web.1.vedbrol0piztelokq42yby1fa

###vCon03 (@ 10.0.2.113) - WORKER NODE

    vCon03:~$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    abacc1fdf489f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 29 seconds ago  Up 28 seconds   80/tcp  03-services_web.5.j3xhf9f3eylr8a20g4emzatu0
    7de1456c4870f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 29 seconds ago  Up 27 seconds   80/tcp  03-services_web.3.ogxzbvhaf79oeumw7zkug3nw3


##CHECK APPLICATION

###vCon01 (@ 10.0.2.111) - SWARM MANAGER

    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> ce1cbc1b49ec<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 95983640e20d<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 7de1456c4870<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> abacc1fdf489<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 0f2af86b6b1e<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    vCon01:~/docker/03-services$ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> ce1cbc1b49ec<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>

*^^Notice the Hostname changes (round-robins) between the 5 containers load-balanced across 3 SWARM Nodes*

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
    
From <https://docs.docker.com/get-started/part3/#recap-and-cheat-sheet-optional> 

###Terminal Recording

https://asciinema.org/a/113837