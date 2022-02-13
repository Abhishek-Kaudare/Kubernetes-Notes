# **Configuration**

## **Docker Basics**

### **Basic Docker File with CMD**
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


### **Docker build**
```bash
docker build -t node_launch .

```

- This command used the Dockerfile to build a new container image. This is because we instructed the builder that we wanted to start from the `node:12-alpine` image. But, since we didn’t have that on our machine, that image needed to be downloaded.
  - After the image was downloaded, we copied in our application and used yarn to install our application's dependencies. 
- **The CMD directive specifies the default command to run when starting a container from this image.**
- Finally, the `-t` flag tags our image. Think of this simply as a human-readable name for the final image. Since we **named the image** `node_launch`, we can refer to that image when we run a container.
- The `.` at the end of the docker build command tells that Docker should look for the Dockerfile in the current directory.



### **Docker Run**
Start container using the `docker run` command and specify the name of the image.
```bash
docker run node_launch
```

#### **Overide CMD in commandline** 
```bash
docker run node_launch node src/index2.js
```

### **Using ENTRYPOINT in dockerfile**

**ENTRYPOINT** sets a default application to be used every time a container is created with the image.

**CMD** command can be used to mention arguments which could be override from commandline.

```dockerfile
FROM node:12-alpine 
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
ENV foo /bar
RUN yarn install --production

# ENTRYPOINT sets a default application to be used every time a container is created with the image.
ENTRYPOINT ["node"]

# CMD command can be used to mention arguments which could be override from commandline.
CMD ["src/index.js"]
```
#### **Overide CMD in commandline** 
```bash
docker run node_launch src/index2.js
```

#### **Overide CMD and ENTRYPOINT in commandline** 
```bash
# Overide to get node version
docker run --entrypoint node node_launch –version
```

## **Pod Configuration comparison with *docker cli***

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      command: ["sleep"] # --entrypoint flag of docker
      args: ["10"] # --cmd of docker
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
```
