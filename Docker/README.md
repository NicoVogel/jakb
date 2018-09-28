# Docker

Docker is a strong tool to quickly setup entire environments and it has a great flexebility.

The topics I want to share are the following:
- Clear Docker
- Dockerfile
- Interation between Container/Volume/Images

# Clear Docker

> the commands are for ubunto, therefore sudo is included. If you do not use ubuntu, you dont need sudo.

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

A dockerfile does contain the information how a image is constructed.

TODO
- commands
- hint: less commands

# Interation between Container/Volume/Images

The following picture illustrates the interactions between the 'main components'.

![Conatiner, Volume, Images interation](./images/Docker_Volume_Conatiner_Image_Interation.svg)

> The icons come from *www.flaticon.com*

Lets discuss the image from left to right. The start of everything is the **Dockerfile**. Images are created out of a dockerfile and every command in the dockerfile does create a **Layer** of an image.

**Images** do consist out of multiple Layers. Every layer does repesent an intermediate build state of the image. So that if one command changes, not the entire image has to be recreated. 