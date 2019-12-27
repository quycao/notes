# Docker Notes

## 1. Commands
- Pull an image from Docker hub
    > docker pull image_name:tag_name
- Build an image from Dockerfile
    > docker build -t image_name:tag_name -f dockerfile_path .
- Create new container (add `-d` flag to make diamon process)
    > docker run -v <folder_in_host>:<folder_in_container> -p <port_in_host>:<port_in_container> -it <image_name> /bin/bash
- Save and load docker image
    > docker save image_repository:tag > ~/Downloads/archive_image.tar
    
    > docker load < ~/Downloads/archive_image.tar
- Other
    ```bash
    docker image ls							    # list installed images
	docker image rm image_id					# remove an image
	docker container ls							# list running container
	docker container ls -a						# list shutdown container
	docker container start container_name		# start a container
    docker container stop container_name        # stop a container
	docker container rm container_id 			# remove a container
	docker exec -it {container_name} /bin/sh	# access to a container, -it: Interactive mode
	docker exec -it {container_name} /bin/bash	# access to a container

	docker rename container_name new_name	    # rename container
	docker tag image_id new_name:tag_name		# rename image repository
	docker system prune						    # remove unused objects (image, container)
    ```

## 2. Creating a Dockerfile
This file acts as a manifest but also as a build instruction file, how to get our app up and running

```docker
# Dockerfile

FROM node:latest                # Selecting an OS image from Docker Hub or local if it was downloaded

WORKDIR /app                    # Set a working directory. This is a way to set up for what is to happen later

COPY . .                        # Copy the files from the directory we are standing into the directory specified by our WORKDIR command

ENV PORT=3000                   # Add an environment variable

RUN npm install                 # This runs a command in the terminal to install/setup somethings in docker image

EXPOSE $PORT                     # Opening up a port, it is through this port that we communicate with our container

ENTRYPOINT ["node", "app.js"]   # This is where we should state how we start up our application, the commands need to be specified as an array so the array [“node”, “app.js”] will be translated to the node app.js in the terminal
```


## 3. Creating an image
Example

```bash
docker build -t quycao/node:latest .
```

Here we omit the dockerfile_path, it will looking for the "Dockerfile" in the current directory. The `.` at the end is important as this instructs Docker and tells it where your Dockerfile is located, it's the path that we specify of the current directory.


## 4. Creating a container
Example

```bash
docker run -p 8000:3000 chrisnoring/node
```


## 4. Improving our set up with Environment Variables
