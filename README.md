# docker-cheatsheet
1. [Project Structure](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#the-project-structure)
2. [Docker](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#docker)
   1. [Main elements to know](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#main-elements-to-know)
   2. [Installation](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#installation)
   3. [Dockerfile components](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#dockerfile-components)
   4. [Key Docker commands](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#key-docker-commands)
   5. [Docker Compose Elements](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#docker-compose-elements)
   6. [Docker Flags](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#docker-flags)
   7. [Volume Types](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#volume-types)
   8. [Restart Policies](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#restart-policies)
3. [Add Docker To Project](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#add-docker-to-the-project)
   1. [Requirements](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#requirements)
   2. [Poll Dockerfile](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#poll-dockerfile)
   3. [Result Dockerfile](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#result-dockerfile)
   4. [Worker Dockerfile](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#worker-dockerfile)
   5. [Dockerignore file](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#dockerignore-file)
   6. [Docker Compose File](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#docker-compose-file)
   7. [Redis service](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#redis)
   8. [DB Service](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#db)
   9. [Worker, Poll and Result services](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#worker-poll-result)
   10. [Networks](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#networks)
   11. [Volumes](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#volumes)
   12. [.env and .gitignore](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#env-and-gitignore)
4. [Run the project](https://github.com/yanamlnk/docker-cheatsheet?tab=readme-ov-file#run-the-project) 

## The project structure
All project files were given by the school (except Docker). Here is a small description:
- it is a web application that has a poll part - where you can vote - and result part - where you can see the results of the vote
- **poll** - written in Python Flask, it pushes the results of the vote to the Redis queue
- **redis** - holds the results until worker consumes and process them
- **worker** - Java application that consumes votes and saves them to PostgreSQL database
- **database** - persistently stores the results
- **result** - Node.js application that fetches the results from database and displays them. Uses Socket.io

So the general schema for infrastructure is the following:
```
  [Poll]                [Result]     <- (frontend)
     |                     |
  [Redis] - [Worker] - [Database]    <- (backend)
```
So there are 5 microservices that needs to be connected and "communicate" with each other.
The final project tree will be the following: 
```
.
├── compose.yml
├── poll
│   ├── Dockerfile
│   └── (poll python files)
├── result
│   ├── Dockerfile
│   └── (result JS files)
├── schema.sql
└── worker
    ├── Dockerfile
    └── (worker Java files)
```
So there will be 3 different containers: poll, result and worker, and a compose file that will connect them all together 

## Docker
There is always a problem, that the same code will be working perfectly on one machine, and giving errors on the other. It happens due to different OS, versions of libraries, etc. This is where Docker comes into the game: it creates an isolated environment called **container**, where your code will be running, and since it is coming pre-configured, it will be working everywhere the same.

### Main elements to know:

1. **Dockerfile**
- Text file containing instructions to build a Docker image
- Series of commands that Docker executes to create an image
- Each instruction creates an immutable layer in the image
- Base component for creating reproducible builds

2. **Docker Image**
- A read-only template containing application code, libraries, dependencies, tools, and other files
- Like a "snapshot" or blueprint for creating containers
- Can be stored in registries (like Docker Hub)
- Built using a Dockerfile
- Layered architecture (each instruction creates a new layer)

3. **Docker Container**
- A runnable instance of a Docker image
- Isolated environment with its own filesystem, network interface, and process space
- Can be started, stopped, moved, and deleted
- Like a lightweight, isolated virtual machine

4. **Docker Volume**
- Mechanism for persisting data generated by and used by containers
- Exists outside the container lifecycle
- Three types:
  - Named volumes (managed by Docker)
  - Bind mounts (direct link to host filesystem)
  - tmpfs mounts (stored in host memory)
- Necessary because data is not persistent in Docker container. Once it is stopped - all data generated will disappear

5. **Docker Network**
- Enables communication between containers
- Isolates container communications
- Types:
  - Bridge (default)
  - Host
  - None
  - Custom networks
 
6. **Docker Registry**
- Storage and distribution system for Docker images
- Can be public (like Docker Hub) or private
- Repository for sharing and versioning images

7. **Docker Compose**
- Tool for defining and running multi-container applications
- Uses YAML file to configure application services
- Manages the complete application lifecycle, sets up networks and control the order of container creations (if there are specific dependencies)

**Relationships**: 
```
Dockerfile -> Docker Image -> Docker Container
                    ^
             Docker Registry

Container <-> Volume (for persistence)
Container <-> Network (for communication)
```

### Installation
Install [Docker Desktop](https://docs.docker.com/desktop/), that includes everything necessary + UI.

To check if you have Docker and Compose: 
- `docker --version`
- `docker compose version`

### Dockerfile components:
```
FROM         # Base image to build upon
WORKDIR      # Sets working directory for instructions
COPY         # Copies files from host to container
ADD          # Copies files (with extra features like URL support and tar extraction)
RUN          # Executes commands during image build
ENV          # Sets environment variables
EXPOSE       # Documents which ports are intended to be published
CMD          # Default command to run when container starts
ENTRYPOINT   # Main command to run (CMD becomes arguments to this)
VOLUME       # Creates a mount point for external volumes
```
### Key Docker commands:
```
# Images
docker build -t name:tag .    # Build image from Dockerfile
docker pull image:tag        # Pull image from registry
docker push image:tag        # Push to registry
docker images               # List local images
docker rmi image           # Remove image

# Containers
docker run image           # Create and start container
docker start/stop name    # Start/stop existing container
docker ps                # List running containers
docker ps -a             # List all containers
docker rm container      # Remove container
docker logs container    # View container logs
docker exec -it container bash  # Enter running container

# System
docker system prune      # Clean up unused resources
docker volume ls        # List volumes
docker network ls      # List networks
```
### Docker Compose Elements:
```
version:        # Compose file version
services:       # Define application services
  webapp:       # Service name
    build:      # Build from Dockerfile
    image:      # Use existing image
    ports:      # Port mapping (host:container)
    volumes:    # Mount volumes
    environment: # Environment variables
    networks:    # Connect to networks
    depends_on:  # Service dependencies
    restart:     # Restart policy

networks:       # Define custom networks
volumes:        # Define named volumes
```
### Docker Flags
```
-d          # Run in background (detached)
-p          # Port mapping
-v          # Volume mounting
--name      # Assign container name
--network   # Connect to network
-e          # Set environment variables
--rm        # Remove container when it exits
-it         # Interactive terminal
```
### Volume Types
```
# Named volumes
volumes:
  mydata:

# Bind mounts
volumes:
  - ./host/path:/container/path

# tmpfs mounts (memory only)
tmpfs:
  - /temp
```

### Restart Policies
```
restart:
  no             # Never restart
  always         # Always restart
  on-failure     # Restart only on failure
  unless-stopped # Always restart unless manually stopped
```

## Add Docker to the project
### Requirements
There are the following requirements: 
- create 3 images, respecting the specifications described below.
- no ENTRYPOINT.
- no latest versions
- Poll
  - the image is based on an official Python image ;
  - the app exposes and runs on port 80 ;
- Result
  - the image is based on an official Node.js Alpine image ;
  - the app exposes and runs on port 80 ;
  - The node_modules folder must be excluded from the build context.
- Worker
  - The image is built using a multi-stage build:
  - First stage - compilation:
    – is based on maven:3.9.6-eclipse-temurin-21-alpine and is named builder.
    – is used to build and package the Worker application using:
      - `mvn dependency:resolve` from within the folder containing pom.xml;
      - then `mvn package` from within the folder containing the src folder.
    – generates a file in the target folder named worker-jar-with-dependencies.jar
  - Second stage - run:
    – is based on eclipse-temurin:21-jre-alpine ;
    – is the one really running the worker using `java -jar worker-jar-with-dependencies.jar`.
- Docker images must be as simple and lightweight as possible.
- Name of the Compose file is `compose.yml`
- Compose file should contain:
  - 5 services:
    – poll (builds poll image, redirects port 5000 of the host to the port 80 of the container)
    – redis (uses an existing official image of Redis, opens port 6379)
    – worker (builds worker image)
    – db (uses an existing official image of PostgreSQL, has its database schema created during container first start)
    – result (builds result image, redirects port 5001 of the host to the port 80 of the container)
  - 3 networks: poll-tier, result-tier and back-tier.
  - 1 volume: db-data.

### Poll Dockerfile
```
# poll/Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
ENV PORT=80
EXPOSE 80
CMD ["python", "app.py"]
```
- **FROM** - takes official Python image with version 3.11. Slim for minimal image size 
- **WORKDIR** - creates a directory INSIDE container (each container has its own filesystem) and sets it as working directory. It is like running command `mkdir /app && cd /app`
- **COPY** - copies files from local directory (first `.`) into the container's current working directory (second `.`). First path to local directory is just `.`, because Dockerfile is already in the `poll` directory will all the source code and configs being inside `poll` directory, too. Command **COPY** includes all files except those in `.dockerignore`
- **RUN** executes specified command during image build. In this case command is  `pip install -r requirements.txt` which is the command to install libraries included in requirements.txt
- **ENV** sets environment variable inside the container. In this case variable **PORT** is created with assigned value **80**
- **EXPOSE** for documentation purposes and is not affecting the container in the global sense. In this case it is exposing port 80 just to put a label that "this container will use port 80". This command itself does not make this port accesible, you need to explicitly publish it in order to use, for example, with command `docker run -p 5000:80 your-image`
- **CMD** is to precise default command to run when container starts. This command can be overridden when starting a container, unlike **ENTRYPOINT** that sets a fixed command that cannot be easily overridden

### Result Dockerfile
```
# result/Dockerfile
FROM node:21-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

ENV PORT=80
EXPOSE 80

CMD ["npm", "start"]
```
Here, everything is the same as in Poll Dockerfile, with a small key difference: there are 2 **COPY**. 
- First **COPY** copies just package.json files and after that install necessary dependencies
- Second **COPY** copies the rest (including `package.json` twice, but it will just rewrite package.json without installing dependencies one more time)
- Why it is done? Docker uses layer caching. If layer created with `COPY package*.json` is not changed, and only code (second **COPY**) is changed - then Docker will copy just the rest of the files and will skip the step with installation of dependencies, saving time. Docker will reinstall dependencies only if `package.json` is changed.
- both `./` and `.` mean "current directory", but:
  - `.` is "everything in current directory"
  - `./` is explicitly "current directory"
 
### Worker Dockerfile
```
# worker/Dockerfile
FROM maven:3.9.6-eclipse-temurin-21-alpine AS builder
WORKDIR /app
COPY . .
RUN mvn dependency:resolve
RUN mvn package

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/worker-jar-with-dependencies.jar .
CMD ["java", "-jar", "worker-jar-with-dependencies.jar"]
```
- this Dockerfile is interesting because it is multi-stage, which is used to create a smaller and more efficient final image.
- First stage:
  - **AS builder** names this stage for later reference
  - **RUN** 2 commands: `mvn dependency:resolve` to download dependencies, `mvn package` to create jar file
- Second stage:
  - **COPY --from=builder** takes this file from the stage named `builder`
 
```
Stage 1 (builder)             Stage 2 (final)
+------------------+          +----------------+
| Maven image      |          | JRE image      |
| Source code      |   JAR    |                |
| Builds JAR    -----→-------→ Only JAR file   |
| ~400MB           |          | ~100MB         |
+------------------+          +----------------+
```
### Dockerignore file
- It tells Docker which files/directories to EXCLUDE during the build process
- Makes builds faster by copying fewer files
- Reduces the final image size

In result folder, I have create `.dockerignore` and added to the file `node_modules`.
Thanks to this:
- Docker skips copying the `node_modules` directory
- Dependencies are cleanly installed inside the container (because they could have been installed using specific OS, which can be unsupposted by other systems)
- Build process is faster and cleaner

### Docker Compose File
Now in the root of the project, create `compose.yml` file:
```
version: '3.8'

services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    networks:
      - poll-tier
      - back-tier
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./schema.sql:/docker-entrypoint-initdb.d/schema.sql
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    networks:
      - back-tier
      - result-tier
    restart: unless-stopped

  worker:
    build: ./worker
    environment:
      - REDIS_HOST=redis 
      - POSTGRES_HOST=db
      - POSTGRES_PORT=${POSTGRES_PORT}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    networks:
      - back-tier
    depends_on:
      - redis
      - db
    restart: unless-stopped
    
  poll:
    build: ./poll
    ports:
      - "5000:80"
    environment:
      - REDIS_HOST=redis 
      - OPTION_A=${OPTION_A}
      - OPTION_B=${OPTION_B}
      - OPTION_C=${OPTION_C}
      - OPTION_D=${OPTION_D}
    networks:
      - poll-tier
    depends_on:
      - redis 
    restart: unless-stopped

  result:
    build: ./result
    ports:
      - "5001:80"
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_PORT=${POSTGRES_PORT}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    networks:
      - result-tier
    depends_on:
      - db
    restart: unless-stopped

networks:
  poll-tier:
  result-tier:
  back-tier:

volumes:
  db-data:
```

- `version` affects available features and syntax and specifies Docker Compose file format version. 3.8 is very stable, even though not the latest one
- `services` defines application containers. The order of service declarations in the `compose.yml` file doesn't determine the startup order. The actual startup order is determined by the `depends_on` configuration.:
### redis:
```
redis:
    image: redis:alpine        # Uses pre-built Redis image
    ports:
      - "6379:6379"           # Port mapping (host:container)
    networks:                  # Connected networks
      - poll-tier
      - back-tier
    restart: unless-stopped
```
- regarding ports: `5000:80` means that outside world uses localhost:5000, inside container uses port 80. When you open `localhost:5000` in browser, Docker forwards that traffic to port 80 in the container
### db:
```
db:
    image: postgres:15-alpine
    volumes:                   # Data persistence
      - db-data:/var/lib/postgresql/data         # Named volume
      - ./schema.sql:/docker-entrypoint-initdb.d/schema.sql  # Init script
    environment:              # Environment variables
      - POSTGRES_USER=${POSTGRES_USER}
```
- environment creates environment values in the container. ${POSTGRES_USER} is a value saved in `.env` file.
- for volumes:
  - `db-data:/var/lib/postgresql/data` creates a named volume in location `/var/lib/postgresql/data`
  - `./schema.sql:/docker-entrypoint-initdb.d/schema.sql` - `./schema.sql` is source file on host machine, `/docker-entrypoint-initdb.d/schema.sql` is a destination path in container. `/docker-entrypoint-initdb.d/` is a special directory in PostdreSQL:
    - PostgreSQL automatically executes any .sql files in this directory
    - Only runs when the database is first initialized (first time startup)
    - Used to set up initial database schema, tables, etc.
- so you don't need to additionaly create a user or configure the database, the container will:
  - Use the credentials from your .env file
  - Automatically create the user
  - Run the schema.sql when the container first starts

### worker, poll, result
```
worker:
    build: ./worker           # Build from Dockerfile
    environment:              
      - REDIS_HOST=redis      # Service discovery
      - POSTGRES_HOST=db      # Reference other services
    depends_on:               # Startup order
      - redis
      - db
```
- `build` includes path to the corresponding Dockerbuild file
- environment values in this case (`redis` and `db`) are the name of the services. Services on the same network can talk to each other using service names as hostnames. Example: Worker can reach Redis using just "redis" as hostname. Docker's internal DNS automatically resolves these service names to the correct container IP addresses. So `REDIS_HOST=redis` tells the app to look for a host named 'redis'
- `depends_on` declares an order. Worker must be created after `redis` and `db`

### networks
```
networks:
  poll-tier:    # For poll and redis
  result-tier:  # For result and db
  back-tier:    # For worker, redis, and db
```
is a declaration for the networks

### volumes
```
volumes:
  db-data:      # Named volume for database persistence
```
is a declaration of the volumes 
  
### .env and .gitignore
In root, there is also `.env` file with all values for environment values. 
```
# Database settings
POSTGRES_USER=____
POSTGRES_PASSWORD=____
POSTGRES_DB=____
POSTGRES_PORT=____

# Vote options
OPTION_A=____
OPTION_B=____
OPTION_C=____
OPTION_D=____
```

And `.gitignore` includes files that are not needed to be pushed to the git: 
```
.env
result/node_modules
worker/target
```
## Run the project
Now to start the project, open the Docker Desktop to start the Docker, and then run command `docker compose up --build`.
You don't need to use `--build` everytime to launch the project, only the for the first time.
To stop container: `docker compose down`.

The application should be accessible at:
- Poll interface: http://localhost:5000
- Results interface: http://localhost:5001


