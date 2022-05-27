# **Docker Basics**

## **Basic Docker File with CMD**
```dockerfile
# FROM defines the base image used to start the build process.
FROM node:12-alpine 
# RUN is the central executing directive for Dockerfiles.
RUN apk add --no-cache python2 g++ make
# The WORKDIR command is used to define the working directory of a Docker container at any given time. 
WORKDIR /app
# Copy files
COPY . .
# ENV sets environment variables.
ENV foo /bar
RUN yarn install --production
# CMD can be used for executing a specific command within the container.
CMD ["node", "src/index.js"]
```


## **Docker build**
```bash
docker build -t node_launch .
```

- This command used the Dockerfile to build a new container image. This is because we instructed the builder that we wanted to start from the `node:12-alpine` image. But, since we didn’t have that on our machine, that image needed to be downloaded.
  - After the image was downloaded, we copied in our application and used yarn to install our application's dependencies. 
- **The CMD directive specifies the default command to run when starting a container from this image.**
- Finally, the `-t` flag tags our image. Think of this simply as a human-readable name for the final image. Since we **named the image** `node_launch`, we can refer to that image when we run a container.
- The `.` at the end of the docker build command tells that Docker should look for the Dockerfile in the current directory.



## **Docker Run**
Start container using the `docker run` command and specify the name of the image.
```bash
docker run node_launch
```

### **Overide CMD in commandline** 
```bash
docker run node_launch node src/index2.js
```

## **Using ENTRYPOINT in dockerfile**

**ENTRYPOINT** sets a default application to be used every time a container is created with the image.

**CMD** command can be used to mention arguments which could be override from commandline.

```dockerfile
FROM node:12-alpine 
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
ENV APP_COLOR blue
RUN yarn install --production

# ENTRYPOINT sets a default application to be used every time a container is created with the image.
ENTRYPOINT ["node"]

# CMD command can be used to mention arguments which could be override from commandline.
CMD ["src/index.js"]
```


## **CMD and ENTRYPOINT**

### **Overide CMD in docker commandline** 
```bash
docker run node_launch src/index2.js
```

### **Overide CMD and ENTRYPOINT in docker commandline** 
```bash
# Overide to get node version
docker run --entrypoint node node_launch –version
```

## **Security in Docker** 

We have learned that unlike virtual machines containers are not completely isolated from their host.

**Containers and the host shared the same kernel.**


Containers are isolated using ***Namespaces*** in Linux. The host has a Namespace and the containers have their own Namespace. All the processors run by the containers are in fact run on the host itself but in their own namespace. 

### **Process Isolation**

- As far as the docker container is concerned, it is in its own namespace and it can see its own processes only. It cannot see anything outside of it or in any other namespace.

- So when you list the processes from within the docker container you see the sleep process with a process ID of one. 
  
- For the docker host all processes of its own as well as those in the child Namespaces are visible as just another process in the system.
  
- So when you list the processes on the host, you see a list of processes including the sleep command but with a different process ID.
  
- This is because the processes can have different process IDs in different namespaces and that's how Docker isolates containers within the system.

### **Users in Docker Security Context**
  
- The docker host has a set of users, a root user as well as a number of non-root users. By default, Docker runs processes within containers as the root user.

- This can be seen in the output of the commands we ran earlier. Both within the container and outside the container on the host the process is run as the root user.

- Now if you do not want the process within the container to run as the root, user you may set the user using the user option within the docker run command and specify the new user ID.

```bash
docker run --user=1000 ubuntu sleep 3600
```
- Another way to enforce user security is to have this defined in the docker image itself at the time of creation. 
  
*For example, we will use the default ubuntu image and set the user id to 1000 using the user instruction. Then build the custom image.*

```dockerfile
# FROM defines the base image used to start the build process.
FROM ubuntu

USER 1000
```

```bash
docker build -t my-ubuntu-image .
```
*We can now run this image without specifying the user id and the process will be run with the user id 1000.*
```bash
docker run my-ubuntu-image sleep 3600
```

`Docker implements a set of security features that limits the abilities of the root user within the container. So the root user within the container isn't really like the root user on the host.`

-  Docker uses Linux capabilities to implement this.

#### **Capablities of root users**
- The root user can literally do anything and so does a process run by the root user.
  
- It has unrestricted access to the system and can perform tasks like:
  - modifying files and permissions on files, 
  - access control
  - creating or killing processes
  - setting group ID or user ID
  - performing network related operations such as 
    - binding to network ports
    - broadcasting on a network control network ports
  - system related operations like 
    - rebooting the host
    - manipulating system clock
  - And many more.

- All of these are different capabilities on a Linux system and you can see a full list at this location: `/usr/include/linux/capability.h`

#### **Capablities of root users on containers**

- You can now control and limit what capabilities are made available to a user. By default docker runs a container with a limited set of capabilities.
  
- And so the processes running within the container do not have the privileges to say reboot the host or perform operations that can disrupt the host or other containers running on the same host.
  
- If you wish to override this behavior and *provide additional privileges* then what is available, use the `cap-add` option in the docker run command.
  
```bash
docker run --cap-add MAC_ADMIN ubuntu
```

- Similarly you *can drop privileges* as well using the `cap-drop` option.
  
```bash
docker run --cap-drop KILL ubuntu
```
  
- In case you wish to run the container with *all privileges enabled* use the `privileged` flag. 
  
```bash
docker run --privileged ubuntu
```