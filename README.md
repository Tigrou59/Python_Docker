**A quickly Python's environement with Docker for Dev and Test under Linux (a lot of versions Python of images Docker for are available)**

**Install Docker CE on Linux "Ubuntu"**
https://docs.docker.com/install/linux/docker-ce/ubuntu/

sudo apt-get update && apt-get install docker-ce docker-ce-cli containerd.io

**Install Compose on Linux systems**

    Run this command to download the latest version of Docker Compose:

sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

    Apply executable permissions to the binary:

sudo chmod +x /usr/local/bin/docker-compose


**Verify version for Docker & Docker Composer**

root@jma-VirtualBox:~/docker_lab/python-docker# docker --version
Docker version 18.06.1-ce, build e68fc7a

root@jma-VirtualBox:~/docker_lab/python-docker# docker-compose --version
docker-compose version 1.17.1, build unknown

**Create a Dockerfile from the repo GitHub :**

curl -L
https://github.com/docker-library/python/blob/3189e185470f8abd8957c78973cda6b2413ca0fe/3.7/stretch/slim/Dockerfile > Dockerfile

**Build an image named Python slim 3.7 from a Dockerfile**
*The build's context is set where is the file Dockerfile created above*

docker image build -t slim/python:3.7 .

**A little screen showing the end of the image's build...**
+ pip --version
pip 19.0.2 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)
+ find /usr/local -depth ( ( -type d -a ( -name test -o -name tests ) ) -o ( -type f -a ( -name *.pyc -o -name *.pyo ) ) ) -exec rm -rf {} +
+ rm -f get-pip.py
Removing intermediate container ded4d017c96c
 ---> 75d9999bdbd0
Step 11/11 : CMD ["python3"]
 ---> Running in 4f24a72e183a
Removing intermediate container 4f24a72e183a
 ---> 717c4c493d2f
Successfully built 717c4c493d2f
Successfully tagged slim/python:3.7

**List the Docker images created with the cbuild command**
root@jma-VirtualBox:~/docker_lab/python-docker#

 docker image ls

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    slim/python         3.7                 717c4c493d2f        2 minutes ago       143MB
    debian              stretch-slim        9a4a82cec2d2        7 days ago          55.3MB

#Now we can run a container named "my_python" on the image named "slim-python:3.7"

docker container run -it --rm --name my-python slim/python:3.7

#For to keep data persistent, we will use a volume Docker mounted on the host and a specific workdir in the container

docker container run -it --rm --name my-python -v "$PWD":/usr/src/myapp -w /usr/src/myapp slim/python:3.7

#Some test for verifying the good functionnalities in our Python container

Python 3.7.2 (default, Feb 13 2019, 10:47:41)
[GCC 6.3.0 20170516] on linux
Type "help", "copyright", "credits" or "license" for more information.

    >>> a, b, c = 1, 1, 1    # fibonacci : a et b servent au calcul des termes successifs, c est un simple compteur
    >>> while c<15:
    ...     print(b, end=' ')
    ...     a, b, c = b, a+b, c+1
    ...
    1 2 3 5 8 13 21 34 55 89 144 233 377 610

The best way is to use Docker Compose for create a service Python 3.7

mkdir compose

cd compose

vi docker-compose.yml

    version: '3'
    services:
      slim-python:
        image: slim/python:3.7
        container_name: python-37
        working_dir: /usr/src/myapp
        stdin_open: true
        tty: true
        command: python3
        volumes:
          - PythonVol:/usr/src/myapp
    volumes:
      PythonVol:

#We have added some option related with the interactive mode, we run the service directly, not in background >> (docker-compose up -d)
#but simply with the run option on the compose service
docker-compose run slim-python

root@jma-VirtualBox:~/docker_lab/python-docker/compose#

docker-compose run slim-python

    Creating network "compose_default" with the default driver
    Creating volume "compose_PythonVol" with default driver
    Python 3.7.2 (default, Feb 13 2019, 10:47:41)
    [GCC 6.3.0 20170516] on linux
    Type "help", "copyright", "credits" or "license" for more information.

    >>>
    CTRL+d

#Verifiing the compose's service
docker-compose ps

    Name   Command   State   Ports
    -----------------------------------
    python-37   python3   Up    

#Now, we can easily use a Docker'command and we can list, inspect the mount point or delete the binding volume Docker
root@jma-VirtualBox:~/docker_lab/python-docker/compose#

    docker volume ls
    DRIVER              VOLUME NAME
    local               compose_PythonVol

docker volume inspect -f '{{.Mountpoint}}' compose_PythonVol
/var/lib/docker/volumes/compose_PythonVol/_data