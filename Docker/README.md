[Go To Overview](../../../)

# Docker

Docker is a strong tool to quickly setup entire environments and it has a great flexibility.

The topics I want to share are the following:
- Clear Docker
- Dockerfile
- Interation between Container/Volume/Images

# Clear Docker

> the commands are for ubuntu, therefore sudo is included. If you do not use ubuntu, you dont need sudo.

**stop all images (kill or stop)**

````shell
sudo docker stop $(sudo docker ps -a -q)
sudo docker kill $(sudo docker ps -a -q)
````

**remove all conatiner**

````shell
sudo docker rm $(sudo docker ps -a -q)
````

**remove all images**

````shell
sudo docker rmi $(sudo docker images -q)
````

**remove all volumes**

````shell
sudo docker volume rm $(sudo docker volume ls -q)
````

# Dockerfile

A dockerfile contains the information how a image is constructed.
The following table is an overview about some commands:

command|usage|wiki link
---|---|---
**FROM** < image name >|the first command in a file which determins the starting point|[link](https://docs.docker.com/engine/reference/builder/#from)
**RUN** < command >|the command will be executed on the image|[link](https://docs.docker.com/engine/reference/builder/#run) 
**COPY/ADD** < host file > < container file>|copy a file into the conatiner to a spesific path with a specific name (ADD and COPY are the same)|[link](https://docs.docker.com/engine/reference/builder/#add)
**CMD** ["< executable >", "< param >", ...]|defines default values for container start (only the last CMD command is executed)|[link](https://docs.docker.com/engine/reference/builder/#cmd)
**EXPOSE** < port >|tells docker, that the container listens to that port|[link](https://docs.docker.com/engine/reference/builder/#expose)

Every RUN command within the Dockerfile creates a layer in an image and therefore the image gets bigger. So you should combine **RUN** commands with **&&** to create less layers to reduce the size.

**Build Dockerfile**

Build an image from a Dockerfile with the following command ([link to the docker wiki](https://docs.docker.com/engine/reference/commandline/build/#parent-command)):

````shell
sudo docker build [options] .
# this command works if the Dockerfile has the name 'Dockerfile'
# options
#   -t <image name[:tag name]> # if tag name is not set ist 'latest'
#   --rm #removes containers that were created for the build process
````

# Interation between Container/Volume/Images

The following picture illustrates the interactions between the 'main components'.

![Conatiner, Volume, Images interation](./images/Docker_Volume_Conatiner_Image_Interation.svg)

> The icons come from *www.flaticon.com*, except for the cylinder

Lets discuss the image from left to right. The start of everything is the **Dockerfile**. Images are created out of a dockerfile and every command in the dockerfile creates a **Layer** of an image.

<u>**Images**</u> consists of multiple layers. Every layer repesents an intermediate build state of an image. So that if one command changes within the *Dockerfile*, not the entire image has to be rebuild.

Every Layer is hashed with *SHA256* and the first 10 digits **(!!! need to check that)** are the visible *IMAGE ID* if you use the comment:

````shell
sudo docker images
````

By using the command above, you do see the image id of the top layer from the image. This is more or less a nice to know fact, but not noticable while using docker. The awareness is important, because of these layers the image size can increes drasticly.

Create a <u>**Container**</u> of an image by using the *run* command ([Link to the docker wiki](https://docs.docker.com/engine/reference/commandline/run/)):

````shell
sudo docker run [options] <image-name>
# options:
#   -h <container name>
#   -v <host directory/file>:<container directory/file>
#   -p <host port>:<container port>
#   -e <environment variable>
#   -d #run in background
````

<u>**Volumes**</u> are created by the container. If you want to persist data you needed to use the **-v** option when running a container. Every directory or file which is mapped with -v is persisted and will be availiable again if the container is restarted.

It does also create volumns for every **VOLUME** command in the Dockerfile. All volumes can be listed with the command: 

````shell
sudo docker volume ls
````

