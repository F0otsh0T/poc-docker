## **Your new development environment** ##

In the past, if you were to start writing a Python app, your first order of business was to install a Python runtime onto your machine. But, that creates a situation where the environment on your machine needs to be perfect for your app to run as expected, and also needs to match your production environment.

With Docker, you can just grab a portable Python runtime as an image, no installation necessary. Then, your build can include the base Python image right alongside your app code, ensuring that your app, its dependencies, and the runtime, all travel together.

These portable images are defined by something called a ***Dockerfile***.


## **Define a container with Dockerfile** ##

***Dockerfile*** defines what goes on in the environment inside your container. Access to resources like networking interfaces and disk drives is virtualized inside this environment, which is isolated from the rest of your system, so you need to map ports to the outside world, and be specific about what files you want to “copy in” to that environment. However, after doing that, you can expect that the build of your app defined in this ***Dockerfile*** behaves exactly the same wherever it runs.

***Dockerfile***

Create an empty directory. Change directories (cd) into the new directory, create a file called ***Dockerfile***, copy-and-paste the following content into that file, and save it. Take note of the comments that explain each statement in your new ***Dockerfile***.

    # Use an official Python runtime as a parent image
    FROM python:2.7-slim
    
    # Set the working directory to /app
    WORKDIR /app
    
    # Copy the current directory contents into the container at /app
    ADD . /app
    
    # Install any needed packages specified in requirements.txt
    RUN pip install --trusted-host pypi.python.org -r requirements.txt

    ## If behind proxy, pip will fail, you must specify proxy in pip cmd, example;
    ## RUN pip --proxy http://135.28.13.11:8080 install --trusted-host pypi.python.org -r requirements.txt

    # Make port 80 available to the world outside this container
    EXPOSE 80
    
    # Define environment variable
    ENV NAME World
    
    # Run app.py when the container launches
    CMD ["python", "app.py"]


**Are you behind a proxy server?**

Proxy servers can block connections to your web app once it’s up and running. If you are behind a proxy server, add the following lines to your ***Dockerfile***, using the ENV command to specify the host and port for your proxy servers:

    # Set proxy server, replace host:port with values for your servers
    ENV http_proxy host:port
    ENV https_proxy host:port
    Add these lines before the call to pip so that the installation succeeds.

This ***Dockerfile*** refers to a couple of files we haven’t created yet, namely app.py and requirements.txt. Let’s create those next.


The app itself

Create two more files, requirements.txt and app.py, and put them in the same folder with the ***Dockerfile***. This completes our app, which as you can see is quite simple. When the above ***Dockerfile*** is built into an image, ***app.py*** and ***requirements.txt*** is present because of that ***Dockerfile***’s ***ADD*** command, and the output from ***app.py*** is accessible over HTTP thanks to the ***EXPOSE*** command.

**requirements.txt**

    Flask
    Redis

**app.py**

from flask import Flask
from redis import Redis, RedisError
import os
import socket

    # Connect to Redis
    redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
    
    app = Flask(__name__)
    
    @app.route("/")
    def hello():
    try:
    visits = redis.incr("counter")
    except RedisError:
    visits = "<i>cannot connect to Redis, counter disabled</i>"
    
    html = "<h3>Hello {name}!</h3>" \
       "<b>Hostname:</b> {hostname}<br/>" \
       "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)
    
    if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)

Now we see that ***pip install -r requirements.txt*** installs the Flask and Redis libraries for Python, and the app prints the environment variable ***NAME***, as well as the output of a call to ***socket.gethostname()***. Finally, because Redis isn’t running (as we’ve only installed the Python library, and not Redis itself), we should expect that the attempt to use it here fails and produces the error message.

Note: Accessing the name of the host when inside a container retrieves the container ID, which is like the process ID for a running executable.

That’s it! You don’t need Python or anything in ***requirements.txt*** on your system, nor does building or running this image install them on your system. It doesn’t seem like you’ve really set up an environment with Python and Flask, but you have.


## Build the app ##

We are ready to build the app. Make sure you are still at the top level of your new directory. Here’s what ls should show:

    $ ls
    Dockerfile		app.py			requirements.txt

Now run the build command. This creates a Docker image, which we’re going to tag using -t so it has a friendly name (in this case, we are calling the container image "friendlyworld").

    docker build -t friendlyhello .

Where is your built image? It’s in your machine’s local Docker image registry:

    $ docker image ls
    REPOSITORY  TAG IMAGE IDCREATED SIZE
    friendlyhello   latest  f8de8ba5f6fa16 minutes ago  148MB
    python  2.7-slim52ad41c7aea45 days ago  139MB


## Run the app ##

Run the app, mapping your machine’s port 4000 to the container’s published port 80 using -p:

    docker run -p 4000:80 friendlyhello

OR

    docker run -d -p 4000:80 friendlyhello

"-d" will run it in "detached" mode so the container will continue running in the background.

You should see a message that Python is serving your app at ***http://0.0.0.0:80***. But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000, making the correct URL ***http://localhost:4000***.

Go to that URL in a web browser to see the display content served up on a web page.



Note: If you are using Docker Toolbox on Windows 7, use the Docker Machine IP instead of ***localhost***. For example, ***http://192.168.99.100:4000/***. To find the IP address, use the command docker-machine ip.

You can also use the curl command in a shell to view the same content.

    $ curl http://localhost:4000
    
    <h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>

This port remapping of ***4000:80*** is to demonstrate the difference between what you ***EXPOSE*** within the ***Dockerfile***, and what you publish using ***docker run -p***. In later steps, we just map port 80 on the host to port 80 in the container and use ***http://localhost***.

Hit ***CTRL+C*** in your terminal to quit.

Now let’s run the app in the background, in detached mode:

    docker run -d -p 4000:80 friendlyhello

You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with docker container ls (and both work interchangeably when running commands):

    $ docker container ls
    CONTAINER IDIMAGE   COMMAND CREATED STATUS  PORTS  NAMES
    41f36ed5f5c4friendlyhello   "python app.py" 21 minutes ago  Up 21 minutes   0.0.0.0:4000->80/tcp   modest_shirley

Notice that CONTAINER ID matches what’s on ***http://localhost:4000***.

Now use ***docker container stop*** to end the process, using the ***CONTAINER ID***, like so:

    docker container stop 41f36ed5f5c4


## Share your image ##

To demonstrate the portability of what we just created, let’s upload our built image and run it somewhere else. After all, you need to know how to push to registries when you want to deploy containers to production.

A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The ***docker*** CLI uses Docker’s public registry by default.

Note: We use Docker’s public registry here just because it’s free and pre-configured, but there are many public ones to choose from, and you can even set up your own private registry using Docker Trusted Registry.

**Log in with your Docker ID**

If you don’t have a Docker account, sign up for one at cloud.docker.com. Make note of your username.
Log in to the Docker public registry on your local machine.

    $ docker login

**Tag the image**

The notation for associating a local image with a repository on a registry is ***username/repository:tag***. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as ***tutorial:02-containers.friendlyhello***. This puts the image in the get-started repository and tag it as ***part2***.

Now, put it all together to tag the image. Run ***docker tag image*** with your username, repository, and tag names so that the image uploads to your desired destination. The syntax of the command is:

    $ docker tag image username/repository:tag

For example:

    $ docker tag friendlyhello username/tutorial:02-containers.friendlyhello
    
    $ docker tag friendlyhello username/tutorial:latest

Run ***docker image ls*** to see your newly tagged image. (You can also use docker image ls.)

    $ docker image ls
    REPOSITORY  TAG IMAGE IDCREATED SIZE
    username/tutorial   02-containers.friendlyhello f8de8ba5f6fa21 minutes ago  148MB
    friendlyhello   latest  f8de8ba5f6fa21 minutes ago  148MB
    python  2.7-slim52ad41c7aea45 days ago  139MB
    ...


**Publish the image**

Upload your tagged image to the repository:

    $ docker push username/repository:tag
    
    $ docker push username/tutorial:02-containers.friendlyhello
    
    $ docker push username/tutorial:latest

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you see the new image there, with its pull command.

Pull and run the image from the remote repository

From now on, you can use docker run and run your app on any machine with this command:

    $ docker run -p 4000:80 username/repository:tag

If the image isn’t available locally on the machine, Docker pulls it from the repository.

    $ docker run -d -p 4000:80 username/tutorial:02-containers.friendlyhello
    Unable to find image 'username/tutorial:02-containers.friendlyhello' locally
    02-buildapp.friendlyhello: Pulling from username/tutorial
    d2ca7eff5948: Already exists 
    cef69dd0e5b9: Already exists 
    50e1d7e4f3c6: Already exists 
    861e9de5333f: Already exists 
    1a23b876549e: Already exists 
    84b1241d9d0e: Already exists 
    ee57a9f67012: Already exists 
    Digest: sha256:4f6035233d92ade938329da6d437977db3550899fc0fc5b3571af6a0338ea132
    Status: Downloaded newer image for username/tutorial:02-containers.friendlyhello
    a45ebfa2cd9556e19e63b34a4c16b60ffc4c9e097841263b78b075b34e6da626

    $ docker ps
    CONTAINER IDIMAGE COMMAND CREATED STATUS  PORTS  NAMES
    a45ebfa2cd95username/tutorial:02-containers.friendlyhello   "python app.py" 5 seconds ago   Up 4 seconds0.0.0.0:4000->80/tcp   optimistic_yalow

No matter where ***docker run*** executes, it pulls your image, along with Python and all the dependencies from ***requirements.txt***, and runs your code. It all travels together in a neat little package, and you don’t need to install anything on the host machine for Docker to run it.
sudo 