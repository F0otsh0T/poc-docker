## About ##

Service: component building block working with other services to create a Distributed Application. "Containers in production".

- One Image
- Ports
- Replicas (scaling)
- Networking
- Flavor (services::<service name>::deploy::resources::limits::cpus/memory)

## docker-compose.yml File ##

    version: "3"
    services:
      web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
    limits:
      cpus: "0.1"
      memory: 50M
      restart_policy:
    condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
    networks:
      webnet:

From <https://docs.docker.com/get-started/part3/#your-first-docker-composeyml-file> 

This ***docker-compose.yml*** file tells Docker to do the following:

- Pull the image we uploaded in step 2 from the registry.
- Run 5 instances of that image as a service called ***web***, limiting each one to use, at most, 10% of the CPU (across all cores), and 50MB of RAM.
- Immediately restart containers if one fails.
- Map port 80 on the host to ***web***’s port 80.
- Instruct ***web***’s containers to share port 80 via a load-balanced network called ***webnet***. (Internally, the containers themselves publish to ***web***’s port 80 at an ephemeral port.) Define the ***webnet*** network with the default settings (which is a load-balanced overlay network).

## Make sure your hosts have NTP configured ##

```
sudo apt install ntp

Edit /etc/ntp.conf, suggest adding internal NTP servers due to proxy constraints

 Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
# more information.
pool 155.165.201.253 iburst
pool 155.165.201.252 iburst
pool 155.165.201.251 iburst

sudo systemctl restart ntp.service

Verify that ntp is syncing... with ntpq

sudo ntpq -p

     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 155.165.201.253 .POOL.          16 p    -   64    0    0.000    0.000   0.000
 155.165.201.252 .POOL.          16 p    -   64    0    0.000    0.000   0.000
 155.165.201.251 .POOL.          16 p    -   64    0    0.000    0.000   0.000
 ntp.ubuntu.com  .POOL.          16 p    -   64    0    0.000    0.000   0.000
+labtime00.mobil 155.165.204.106  2 u   39   64    3    9.953   -2.087   2.263
*labtime01.mobil 155.165.204.108  2 u   31   64    7   20.305   -0.023   3.694
+labtime02.mobil 155.165.204.106  2 u   68   64    7   18.769    2.946   0.828


```


## Run your new Load-Balanced App ##

    ## Start Docker Swarm
    $ docker swarm init --advertise-addr 192.168.56.111
    Swarm initialized: current node (on9lrmqt72vtb13i9k3f7a98x) is now a manager.
    
    To add a worker to this swarm, run the following command:
    
    docker swarm join --token SWMTKN-1-2yps8ycbs09opuumiy5qzu7vpr4ukrc9p2voqbxk9z60wtprja-cdoq5vq8kxiylyzrfdm5giox1 192.168.56.111:2377
    
    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    
    
    ## Start Application - Give it a NAME (03-services.getstartedlab) 
    $ docker stack deploy -c docker-compose.yml 03-services
    Creating network 03-services_webnet
    Creating service 03-services_web
    
    
    ## Get Service ID for the Service
    $ docker service ls
    ID  NAMEMODEREPLICASIMAGE   PORTS
    r8ov8hg0hsc503-services_web replicated  5/5 f0otsh0t/tutorial:02-containers.friendlyhello   *:80->80/tcp
    
    ## Get Container IDs
    $ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    61b0f8c9bddbf0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 8 minutes ago   Up 7 minutes80/tcp  03-services_web.5.xve64lunbxr3ntjqogjc072qb
    5fa4000058a7f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 8 minutes ago   Up 7 minutes80/tcp  03-services_web.3.vha2mkbuh9c6al9ebwbfp99yt
    aa592ba0a870f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 8 minutes ago   Up 7 minutes80/tcp  03-services_web.4.8orz9qx3zd4gbdz3c3h6fm38e
    2916989d1d76f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 8 minutes ago   Up 7 minutes80/tcp  03-services_web.2.no8w8085yvu2nxmffie39zu42
    0e9d2ad04f02f0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 8 minutes ago   Up 8 minutes80/tcp  03-services_web.1.6kbp26h8g02ruai517cji9n4o
    
    ## Test Service
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 5fa4000058a7<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> aa592ba0a870<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 2916989d1d76<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 61b0f8c9bddb<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 0e9d2ad04f02<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
    $ curl -4 http://localhost
    <h3>Hello World!</h3><b>Hostname:</b> 5fa4000058a7<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>

^^Notice the Hostname changes (round-robins) between the 5 containers

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
    docker service ls # List running services associated with an app
    docker service ps <service>  # List tasks associated with an app
    docker inspect <task or container>   # Inspect task or container
    docker container ls -q  # List container IDs
    docker stack rm <appname> # Tear down an application
    docker swarm leave --force  # Take down a single node swarm from the manager
    
From <https://docs.docker.com/get-started/part3/#recap-and-cheat-sheet-optional> 
