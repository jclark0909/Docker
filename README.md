#Docker

WKDIR = C:\Docker\diveintodocker\src\06-docker-in-the-real-world\03-creating-a-dockerfile-part-1

##BUILDING DOCKER IMAGES

Note specify name and directory you are building the docker file.

-t = tag

docker image build -t web1 .

Inspecting a docker image - .json dump

docker image inspect web1

Adding tag version 

docker image build -t web1:1.0 .

Listing docker images 

docker image ls 

Removing a tagged docker image 

docker image rm web1:1.0

Pushing docker image to docker hub

docker login 
docker image tag web1 jclark0909/web1:latest 
docker image push jclark0909/web1:latest

Deleting image off local 

docker image rm  -f 28db

Pulling docker image from docker bug

docker pull jclark0909/web1:latest


##RUNNING DOCKER CONTAINERS

Creating a tag
docker image tag jclark0909/web1 web1 
tag = web1

Deleting a tag 

docker image rm jclark0909/web1

Listing docker repo's 

docker image ls

List containers 

docker container ls

docker container ls -a 
-a = all stopped containers 

Running container

docker container run -it -p 5000:5000 -e FLASK_APP=app.py web1

-it -allows user input inside container
-p = ports 2 ports 
First port is on the host and second port is on the container
-e = enviroment variable 
web1 = image we want to run inside the container
--rm = automatically going to remove container when it is stopped
--name = name of container
-d = running container is detached mode which means it is running in background

docker container run -it --rm --name web1 -p 5000:5000 -d -e FLASK_APP=app.py web1 

logs = adds logs when running docker container

docker container logs web1

-f = logs in real time 

docker container logs -f web1 

stats = return results in real time about running container(RAM, CPU, etx)

docker container stats

Running a second container in docker
-p = 5001:5000 or -p 5000 - this will allow you to run multple instance of the same container at once
-name web1_2 = You have to change the name of the container in order to run multiple instances

docker container run -it --rm --name web_2 -p 5000 -d -e FLASK_APP=app.py web1

--restart on-failure = allows the container to restart on failure. Helpful in PROD enviroments. Note: You will need to remove the --rm flag.

docker container run -it --name web_3 -p 5000 -d -e FLASK_APP=app.py --restart on-failure  web1

Stopping individual container 

Get list of running containers 

docker container ls 
docker container stop web1 
docker container stop web1_2

Remove a container that is stopped 

 docker rm $(docker ps -a -q)

##LIVE CODE RELOADING WITH VOLUMES

Turning on flash app debug mode 

-e FLASH_DEBUG=1

docker container run -it -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --rm --name web1 web1

Rebuild image to see code change. Modified app.py file to return 'Hello World' Note must be in directory where docker file exsists in order to re-build image. Alternamively you can explicity set where the docker file exsist by replacing the '.' to the docker file location.

docker image build -t web1 .

Mount in the source code to the running docker container by using volumes. This is helpful because we don't have to re-build image each time a code change takes place. 

-v = This mounts a volume from your local machine to a directory on the docker container. Specify local directory first followed by a ':' then the directory on the docker container where you would like the files placed.

docker container run -it -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --rm --name web1 -v C:\Docker\diveintodocker\src\06-docker-in-the-real-world\03-creating-a-dockerfile-part-1:/app web1

Run an interactive session from inside docker container to troubleshoot any issues.

docker container exec -it web1 bash

Note: ctrl + D to close out of bash.

LINKING CONTAINERS WITH DOCKER NETWORKS

List docker networks 

docker network ls

Look at specific network details

docker network inspect bridge

Creating a redis docker container 

docker container run --rm -itd -p 6379:6379 --name redis redis:3.2-alpine

Creating alpine container called web2

docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name web2 -v C:\Docker\diveintodocker\src\06-docker-in-the-real-world\09-linking-containers-with-docker-networks:/app web2

List ip settings for container 

docker exec redis ifconfig


Ping another container from container 

docker exec web2 ping 127.17.0.2

Find container IP 

docker exec redis cat /etc/hosts

Then 

docker container ls - to see if it matches

Creating a new network 

docker network create --driver bridge firstnetwork

Stop containers on default bridge network 

docker container stop redis

docker container stop web2

Creating container on a specific network(firstnetwork)

docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork redis:3.2-alpine 

docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name web2 -v C:\Docker\diveintodocker\src\06-docker-in-the-real-world\09-linking-containers-with-docker-networks:/app --net firstnetwork web2

Verify they are running on firstnetowrk

docker network inspect firstnetwork

Check IP settings for each container 

docker exec web2 ifconfig

docker exec redis ifconfig

Test by pinging container from another container

docker exec web2 ping 172.18.0.2

OR

docker exec web2 ping redis

Stop containers 

docker container stop redis 

docker container stop web2

PRESISTING DATA TO YOUR DOCKER HOST 

Create a named volume

docker volume create web2_redis

List named volumes

docker volume ls 

Inspect volume 

docker volume inspect web2_redis

Re-create the redis container with the mounted named volume 

docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork -v web2_redis:/data redis:3.2-alpine 

##SHARING DATA BETWEEN CONTAINERS

Lecture 11

Start up redis container and web2 container(alpine)

--volumes-from = specifys the volume to use

docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork -v web2_redis:/data --volumes-from web2  redis:3.2-alpine 

docker container run --rm -itd -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name web2 -v C:\Docker\diveintodocker\src\06-docker-in-the-real-world\09-linking-containers-with-docker-networks:/app --net firstnetwork web2

Verify the volume is on the redis box

docker container exec -it redis sh

stop containers 

##RUNNING SCRIPTS WHEN A CONTAINER STARTS

ENTRYPOINT ["..."]

1 image, many projects

ENDPOINT let you run custom scripts after the images is build. This does not add another layer onto the image build step. ETC. Database migrations, ngimnx config

Start up entrypoint container after build(docker image build -it webentrypoint .) and redis container

docker container run --rm -it -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name webentrypoint -v C:\Docker\diveintodocker\src\06-docker-in-the-real-world\13-running-scripts-when-a-container-starts:/app --net firstnetwork webentrypoint

docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork -v web2_redis:/data  redis:3.2-alpine 

Kill webentrypoint container and re-build with a new enviroment variable.



docker container run --rm -it -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 --name webentrypoint -e WEB2_COUNTER_MSG="Docker fans have visited this page" --net firstnetwork webentrypoint



docker container run --rm -itd -p 6379:6379 --name redis --net firstnetwork -v web2_redis:/data  redis:3.2-alpine 


##DOCKER CLEAN UP

Checks disk space for docker images/containers 

docker system df 

Verbose output 

docker system df -v

List of images 

docker image ls

Note:if any images is marked none it is a dangling images and is safe for deletion. 

Docker system information 

docker system info

Clean up old docker images/containers 

docker system prune

-f = force flag 

docker system prune -f 

Remove all unused images(aggressive). This deletes all images with no active containers. 

docker system prune -a 

Stopping more then one container at a time 

docker container stop aaa bbb ccc

OR
-q = quite mode 
docker container stop $(docker container ls -a -q)

##DOCKER COMPOSE

Allows you to build, manage and compile multiple images/containers via a YAML file.



