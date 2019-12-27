# Docker Notes

## 1. Commands
- Pull an image from Docker hub
    ```bash
    docker pull image_name:tag_name
    ```
- Build an image from Dockerfile
    ```bash
    docker build -t image_name:tag_name -f dockerfile_path .
    ```
- Create new container (add `-d` flag to make diamon process)
    ```bash
    docker run -v <folder_in_host>:<folder_in_container> -p <port_in_host>:<port_in_container> --name container_name -it <image_name> /bin/bash
    ```
- Save and load docker image
    ```bash
    docker save image_repository:tag > ~/Downloads/archive_image.tar
    
    docker load < ~/Downloads/archive_image.tar
    ```
- Other
    ```bash
    docker image ls							    # list installed images
	docker image rm image_id					# remove an image
	docker container ls							# list running container
	docker container ls -a						# list shutdown container
	docker container start container_name		# start a container
    docker container stop container_name        # stop a container
	docker container rm container_id 			# remove a container
    docker container logs container_name        # view logs of container
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


## 4. Using a volume
Volumes or data volumes is a way for us to create a place in the host machine where we can write files so they are persisted. Why would we want that? Well, when we are under development we might need to put the application in a certain state so we don’t have to start from the beginning. Typically we would want to store things like log files, JSON files and perhaps even databases (SQLite ) on a volume

- Creating and managing a volume
    ```bash
    docker volume ls

    docker volume create [name_of_volume]

    docker volume rm [name_of_volume]

    docker volume inspect [name_of_volume]    // view volume details

    docker volume prune   // remove volumes that no container used
    ```

- Mounting a volume in your application

    We can use two different commands that achieve relatively the same thing with a different syntax

    - -v, --volume, the syntax looks like the following -v [directory_in_host]:[directory_in_container], for example -v my-volume:/app

    - --mount, the syntax looks like the following--mount source=[directory_in_host],target=[directory_in_container] , for example —-mount source=my-volume,target=/app

    Example:
    ```bash
    docker run -d -p 8000:3000 --name golang_alpine -v /home/admin/src:/logs golang:alpine
    ```

- Treating our application as a volume
    We should mount the our application source code folder to see it on container and make it persisted

    ```bash
    docker run -d -p 8000:3000 --name golang_alpine -v $(PWD):/app golang:alpine
    ```

    > NOTE, if your PWD consists of a directory with space in it you might need to specify the argument as "$(PWD)":/app instead, i.e we need to surround $(PWD) with double quotes.


## 5. Working with databases
- Create a mysql container, it will auto pull down the image for us if we don't have it
    ```bash
    docker run --name mysql_db -e MYSQL_ROOT_PASSWORD=complexpassword -d -p 8000:3306 mysql
    ```

    > We must specify root password for database uninitialized to avoid password specified errors

- Connect to mysql from outside of container
    ```bash
    mysql -uroot -p complexpassword -h 0.0.0.0 -P 8001
    ```
    > We can either specify the password or just write -p and we will be prompted for the password on the next row

- Connect to database from other container
    We must specify IP, port of database in our application
    ```javascript
    // app.js

    const mysql = require('mysql');

    const con = mysql.createConnection({
        host: "localhost",
        port: 8001,
        user: "root",
        password: "complexpassword",
        database: 'Customers'
    });

    con.connect(function (err) {
        if (err) throw err;
        console.log("Connected!");
    });
    ```

- Use network to make containers talk to each other
    - Create the network we want the app container and database container to belong to
        ```bash
        docker network create --driver bridge network_name
        ```

    - Now that we have our network we just need to create each container with the newly created network as an additional argument
        ```bash
        docker run -d -p 8000:3000 --net network_nam --name container_name image_name

        docker run -p 8000:3306 --net network_name --name db_container_name -e MYSQL_ROOT_PASSWORD=complexpassword -d db_image_name
        ```
    Then our application will use the link_alias without to specify IP and port of database
        ```javascript
        // app.js

        const con = mysql.createConnection({
            **host: "db_container_name",**
            user: "root",
            password: "complexpassword",
            database: 'Customers'
        });
        ```

- Linking (old way, create network is prefer)

    The idea of linking is that a container shouldn’t have to know any details on what IP or PORT the database, in this case, is running on. It should just assume that for example the app container and the database container can reach each other. A typical syntax looks like this

    ```bash
    docker run -d -p 5000:5000 --name app_container_name --link db_container_name:link_alias chrisnoring/node
    ```


## 6. Docker Compose

If we have more than two containers the amount of commands we need to type suddenly grows in a linear way.

### Commands
```bash
docker-compose ps           # list running containers
docker-compose build        # build images
docker-compose up           # run containers (ad -d for daemon)
docker-compose down         # stop and remove containers
docker-compose stop         # only stop containers
```

### Docker-compose.yaml
You can configure the following topics inside of a `docker-compose.yaml` file

- Build, we can specify the building context and the name of the Dockerfile, should it not be called the standard name

- Environment, we can define and give value to as many environment variables as we need

- Image, instead of building images from scratch we can define ready-made images that we want to pull down from Docker Hub and use in our solution

- Networks, we can create networks and we can also for each service specify which network it should belong to, if any

- Ports, we can also define the port forwarding, that is which external port should match what internal port in the container

- Volumes, of course, we can also define volumes

Example

```docker
// docker-compose.yaml

version: '3'                        # use latest version
services:
    product-service:                  # app image, container name
        build:
            context: ./product-service    # where our Dockerfile is
        ports:
            - "8000:3000"
        environment:  
            - env_name=testvalue
        volumes:  
            - type: bind                # like 
            source: ./product-service  
            target: /app 
        networks:
            - product-net               # use network name define in root of docker compose file
        depends_on: product-db      # wait for another container to start up first

    inventory-service:
        build:
            context: ./inventory-service
        ports:
            - "8001:3000"
    
    product-db:
        build: ./product-db
        restart: always
        environment:
            - "MYSQL_ROOT_PASSWORD=complexpassword"
            - "MYSQL_DATABASE=Products"
        ports:
            - "8002:3306"
        networks:
            - products-net
    
networks:
    products-net:                   # define a network name
```

Project structure

```
/pet_application
    docker-compose.yml
    /product_service
        Dockerfile
        app.js
        package.json
        wait-for-it.sh
    /inventory_service
        Dockerfile
        app.js
    /product_db
        Dockerfile
        init.sql
```

The corresponds commands

```bash
docker build -t [default name]/product-service .            # the [default name] will be parent_directory by default

docker run -p 8000:3000 --name [default name]/product-service
```

### Networks and databases
- Adding a database
    ```docker
    // docker-compose.yaml

    product-db:
        image: mysql
        environment:
            - MYSQL_ROOT_PASSWORD=complexpassword
        ports:
            - 8002:3306
    ```

- Connecting to database
    - Application and database need to be in the same network. Let's create a network and we place each container in that network.
        ```docker
        // docker-compose.yaml

        services:
            product-service:
                depends_on: product-db      # wait for another container to start up first
                networks:  
                    - products-net              # use network name define in root of docker compose file
            product-db:
                image: mysql

        networks:
            products-net:                       # define a network name
    ```

- Wait for database to initialize fully
    - Docker suggests several scripts for this. All these scripts have one thing in common, the idea is to listen to a specific host and port and when that replies back, then we run our app.
        - wait-for-it
        - dockerize
        - wait-for
    
    - what we need to do
        - Copy this script into your service container. To do that, we simply get it from github and put to application source directory
            ```
            /product-service
                wait-for-it.sh
                Dockerfile
                app.js
                package.json
            ```
        - Give the script execution rights. Let update the `Dockerfile` as below. Worth noting is how we also remove the `ENTRYPOINT` from our `Dockerfile`, we will instead instruct the container to start from the `docker-compose.yaml` file
            ```docker
            // Dockerfile

            FROM node:latest
            WORKDIR /app
            ENV PORT=3000
            COPY . .
            RUN npm install
            EXPOSE $PORT
            RUN chmod +x /app/wait-for-it.sh
            ```

        - Instruct the docker file to run the script with database host and port as args and then to run the service once the script OKs it
            ```docker
            // docker-compose.yaml

            services:
                product-service:
                    command: ["/app/wait-for-it.sh", "product-db:8002", "--", "npm", "start"]   # start node application after get response from port 8002 of db
                product-db:
                // definition of db service below
            ```
        
        - Setting up the database - fill it with structure and data, we must use Dockerfile to build database image
            ```
            // project structure

            /product-service
                some_files
            /product-db
                Dockerfile
                init.sql
            ```

            ```docker
            // docker-compose.yaml

            product-db:
                build: ./product-db
                restart: always
                environment:
                    - "MYSQL_ROOT_PASSWORD=complexpassword"
                    - "MYSQL_DATABASE=Products"
                ports:
                    - "8002:3306"
                networks:
                    - products-net
            ```

            ```docker
            // Dockerfile

            FROM mysql:5.6

            ADD init.sql /docker-entrypoint-initdb.d    # copied init.sql and renamed to docker-entrypoint-initdb.d which means it will run the first thing that happens
            ```

            ```sql
            // init.sql

            CREATE DATABASE IF NOT EXISTS Products;

            # create tables here
            # add seed data inserts here
            ```


        - Modify node app to use database
            ```javascript
            // app.js

            const express = require('express')
            const mysql = require('mysql');
            const app = express()
            const port = process.env.PORT || 3000;
            const test = process.env.test;

            let attempts = 0;

            const seconds = 1000;

            function connect() {
            attempts++;

            console.log('password', process.env.DATABASE_PASSWORD);
            console.log('host', process.env.DATABASE_HOST);
            console.log(`attempting to connect to DB time: ${attempts}`);

            const con = mysql.createConnection({  
            host: process.env.DATABASE_HOST,  
            user: "root",  
            password: process.env.DATABASE_PASSWORD,  
            database: 'Products'  
            });

            con.connect(function (err) {  
            if (err) {  
                console.log("Error", err);  
                setTimeout(connect, 30 * seconds);  
            } else {  
                console.log('CONNECTED!');  
            }

            });

            conn.on('error', function(err) {  
                if(err) {  
                console.log('shit happened :)');  
                connect()  
                }   
            });

            }
            connect();

            app.get('/', (req, res) => res.send(`Hello product service, changed ${test}`))

            app.listen(port, () => console.log(`Example app listening on port ${port}!`))
            ```

## Reference: https://softchris.github.io/pages/docker-one.html
