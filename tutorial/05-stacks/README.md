#Stacks

##Add a new Service, Update, and Re-Deploy

###docker-compose.yml

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
    networks:
      webnet:

From <https://docs.docker.com/get-started/part5/#add-a-new-service-and-redeploy> 

The only thing new here is the peer service to web, named visualizer. Notice two new things here: a volumes key, giving the visualizer access to the host’s socket file for Docker, and a placement key, ensuring that this service only ever runs on a swarm manager – never a worker. That’s because this container, built from an open source project created by Docker, displays Docker services running on a swarm in a diagram.

###Update Stack and Re-Deploy (from SWARM MASTER)

    vCon01:~$ docker stack deploy -c docker-compose.yml 03-services
    Updating service 03-services_web (id: kiqju7nlhmhbjrtkkve7fyg7b)
    Creating service 03-services_visualizer
    
    vCon01:~$ docker service ls
    ID  NAME MODEREPLICASIMAGE   PORTS
    m4k5d8r5dvzf03-services_visualizer   replicated  1/1 dockersamples/visualizer:stable *:8080->8080/tcp
    kiqju7nlhmhb03-services_web  replicated  5/5 f0otsh0t/tutorial:02-containers.friendlyhello   *:80->80/tcp
    
    vCon01:~$ docker ps
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS   NAMES
    d4c07a93f83cdockersamples/visualizer:stable "npm start" 40 seconds ago  Up 39 seconds   8080/tcp03-services_visualizer.1.sx5smxn6hublo94g4jrtngicr
    95983640e20df0otsh0t/tutorial:02-containers.friendlyhello   "python app.py" 2 hours ago Up 2 hours  80/tcp  03-services_web.2.3g5wvfko4to2e7o186aclqgvg

###Verify New Service in Stack

In Browser, go to URL of new Visualizer Service or ***curl -4 http://localhost:8080*** (or whatever you set the URL of the service in your ***docker-compose.yml*** file)

(In my VirtualBox, I have localhost:11188 PORT-FORWARD to 10.0.2.111:8080)


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
