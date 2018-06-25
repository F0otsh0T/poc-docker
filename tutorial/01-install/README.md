# Docker Install

[ONLY IF YOU HAVE PROXY CONSIDERATIONS - SKIP TO DOCKER BASE INSTALL SECTION OTHERWISE]

If your environment requires SSL/HTTPS proxy, you may need to do the following prior to below activities to support proxy with docker:

```
The Docker daemon uses the HTTP_PROXY, HTTPS_PROXY, and NO_PROXY environmental variables in its start-up environment to configure HTTP or HTTPS proxy behavior. You cannot configure
these environment variables using the daemon.json file.
This example overrides the default docker.service file.
If you are behind an HTTP or HTTPS proxy server, for example in corporate settings, you will need to add this configuration in the Docker systemd service file.

1. Create a systemd drop-in directory for the docker service:

    $ mkdir -p /etc/systemd/system/docker.service.d

2. Create a file called /etc/systemd/system/docker.service.d/http-proxy.conf that adds the HTTP_PROXY environment variable:

    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/"

    Or, if you are behind an HTTPS proxy server, create a file called /etc/systemd/system/docker.service.d/https-proxy.conf that adds the HTTPS_PROXY environment variable:

    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443/"

3. If you have internal Docker registries that you need to contact without proxying you can specify them via the NO_PROXYenvironment variable:

    Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"

    Or, if you are behind an HTTPS proxy server:

    Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"

4. Flush changes:

    $ sudo systemctl daemon-reload

5. Restart Docker:

    $ sudo systemctl restart docker

6. Verify that the configuration has been loaded:

    $ systemctl show --property=Environment docker
    Environment=HTTP_PROXY=http://proxy.example.com:80/

    Or, if you are behind an HTTPS proxy server:

    $ systemctl show --property=Environment docker
    Environment=HTTPS_PROXY=https://proxy.example.com:443/

```


### Docker Base Installation: ( https://docs.docker.com/installation )

_NON-UBUNTU INSTALL_:
```
$ sudo -H wget -qO- https://get.docker.com/ | sh
```

***OR***

You may need to find distro-specific install instructions for your environment @:

_UBUNTU INSTALL_:
```       
$ sudo -H apt-get install -y apt-transport-https ca-certificates curl software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

NOTE: If fails, leverage -x proxy options with curl;

curl -x http://one.proxy.att.com:8080 -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo -H apt-get update

$ sudo apt-key fingerprint 0EBFCD88
pub   4096R/0EBFCD88 2017-02-22

      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

$ sudo apt-get update

## TO SEE WHICH VERSIONS ARE AVAIABLE TO INSTALL:
#$ apt-cache madison docker-ce
# docker-ce | 18.03.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 18.03.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.12.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.09.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.09.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.06.2~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.06.1~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.06.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.03.2~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages

$ sudo -H apt-get install docker-ce=17.03.2~ce-0~ubuntu-xenial

## Add your user to execute Docker Commands without needing to SU/SUDO
$ sudo -H usermod -aG docker <username>

```

### Test Installation:

```
$ sudo -H docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b04784fba78d: Pull complete 
Digest: sha256:f3b3b28a45160805bb16542c9531888519430e9e6d6ffc09d72261b0d26ff74f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

$ sudo -H docker version

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
https://cloud.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/engine/userguide/
```

### Install docker-machine (OPTIONAL):

This is used as a "remote" shell to execute docker commands from one node to the docker Daemon on another node.

[WINDOWS]

```
$ if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi && \
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" && \
chmod +x "$HOME/bin/docker-machine.exe"
```

[LINUX::UBUNTU]

```
$ curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && \
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

( https://docs.docker.com/machine/install-machine/#install-machine-directly )

```
$ docker-machine version
docker-machine version 0.13.0, build 9ba6da9
```

### Install docker-compose:

```
$ sudo -H curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$ sudo -H chmod +x /usr/local/bin/docker-compose

$ docker-compose version
docker-compose version 1.19.0, build 9e633ef
docker-py version: 2.7.0
CPython version: 2.7.13
OpenSSL version: OpenSSL 1.0.1t  3 May 2016
```