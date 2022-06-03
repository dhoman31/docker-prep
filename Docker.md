# Override Default Commands

Docker by default will run a default command. ie. `docker run busybox` will run some default command. 

TO pass a custom command simply add this at the end. 

`docker run busybox echo hi there` 
or 
`docker run busybox ls`

It's that simple. 

The command has to be part of the image though in order for this to work. If it is not there then this will not work. 

So what is included in the file image is heavily important. 

# List Running Containers
`docker ps`

Will only show what is running at a moment in time. 

`docker ps --all`

Showcases all of the containers that have ever been ran.

# Container Lifecycle

`docker run` = `docker create` + `docker start`

`create` is just about creating the filesystem (FS)

When doing `docker create hello-world` it will return the container ID. 

Then do `docker start -a <container_id>` will start the container. 

`-a` will print output from the container to the terminal. So it's a more verbose style of starting. 

# Restarting Stopped Containers
When a container stops and you run `docker start -a <container_id>` it will re-run the command you initially gave it (providing you decided to). This means you cannot do something like `docker start -a <container_id> new-command` because Docker will be like "no, you did not originally do this."

# Removing Stopped Containers
`docker ps --all`
to see all. 

To remove all 
`docker system prune`

This will mean you will have to redownload images from Docker hub to get everything again. Can free up a lot of disk space. 

# Retrieving Log Outputs
`docker logs <container_id>`
Just retrieves the logs that are there for the container. 

# Stopping Containers
`docker create busybox ping google.com`
`docker start <container_id`

`docker logs <container_id`

How do we stop the container now? Lets get it to stop itself. 

`docker stop <container id>` -> SIGTERM message is submitted - to stop on it's own merit. Allows it to stop safely. 

`docker kill <container_id>` -> Stop the process right away - you do not get the time to finish your own thing. 

`stop` has 10 seconds to finish its thing before `kill` kicks in. 

Always try to use `stop` first. 

# Multi-Command Containers 
Sometimes we are in scenarios such as with Redis doing something like `redis-server` and then we need to access the `redis-cli` to fetch info. In the case of docker we cannot do that. So we need to pass many commands to do so. 

## Executing Multiple Commands
`docker exec -it <container_id> <command>`
will allow you to run an additional command inside the container. 

Example:
Ensure the `redis` container is running, then in a new window type. 

`docker exec -it acfcd7de143c redis-cli`

if the `-it` flag is not added, it means there is no text input and it will kick us back to the terminal where we cannot enter in anything. 

# IT Flag
The IT flag is 2 flags

`-i` is input -> connects us to STDIN

`-t` -> makes sure the output is formatted well. Very basic view. 

# Getting a Command Prompt in a Container
Get terminal access to the container. 

`docker exec -it <container_id> sh`

We are executing the sh process inside a container. 

# Starting with a shell

To just have a shell and play around with that or use that as a starting point, can do the following: 

`docker run -it busybox sh`

# Container Isolation
Unless a connection between 2 or more containers is made, they are completely independant of one another. You could even be using the same image name but they are treated as different containers. 
# Creating Docker Images
How do we build out own images. It's a very straight process. 

Need to make a Dockerfile

The process is: 

Dockerfile -> Docker Client -> Docker Server -> Usable Image

**Dockerfile:** Commands and processes that need to be done by the container. 

**Docker Server:** Take the file and builds the image to start a new container. 

We need a base image and then some commands to do additional installs, then finally start the container. So 3 parts overall. 

# Building a Dockerfile
Make sure the file is always written as Dockerfile. 

Example below:

```Dockerfile
# Use an existing docker image as a base

FROM alpine

  
  

# Download and install a dependancy

RUN apk add --update redis

  

# Tell the image what to do when it starts as a container

CMD ["redis-server"]
```

After this, run `docker build .`

Then run the container. 

What is `docker build` -> generate an image from a Dockerfile. The output of this command is the build context. 

Essentially each step along the way will generate a new container, the new container or intermediate container will run it's commands, etc. A snapshot is taken which is passed to the next step. The intermediate container is removed so the only container present is the most recent build. 

# Rebuild with cache
If we decided to add an additional line to a Dockerfile, it will use a cache instead as it can see we already used the previous commands before. If nothing has changed since the last time `docker build` was ran, then it will use cache instead since it knows the subsequent steps also will not have changed. 

Only a changed line, down will need to run within the file. If the order of operations change such as lines moving up and down, then it will have to run the whole processes/instructions again. 

Always put changes as far down as possible. 



# Tagging an Image
To add a name to an image, we apply a tag. We will do this with the `-t` command. 


`docker build -t name/image`

So an example:
`-t danielhoman/redis:latest`

So you have a docker ID -> `danielhoman` 

then a `/`

then Repo/Project name -> `redis`

`:` 

and a version -> `latest`

Final command would be `docker build -t danielhoman/redis:latest .`

where `.` is the build context of where we build from. 

Runs with `docker build -t danielhoman/redis` , don't actually need the `latest` at the end. 


# Manually Configuring Images
This can be done using `docker commit` however it is recommended to not go down this path, so should always follow the Dockerfile approach. It's cleaner and easier. 

# Project Outline
Creating a NodeJS app, that we can run through a container and access on our own machine. 

Create a node app, create a dockerfile, build the image, run the image and then use the browser to connect. 

Can be found in here: `/Users/daniel.homan/Desktop/Docker/simpleweb`



# COPY
Add work files to container through the `COPY` command

# Network Mapping
Network mapping information is isolated to each container. Meaning that if we wanted to load a Docker container and access information over a port through a web-browser for example, then port mapping needs to be handled here. This will allow traffic to be received from both host and container. 

Take request from local network and forward to container on that port. 

Need the `-p` flag. 

Command : `docker run -p 8080:8080 <image_id>`
For 8080 is the incoming requests to port on localhost and the second 8080 is the port inside the container. 

Ports also do not need to be identical here. 

# Specifying a working directory
This will prevent any overriding taking place. All working files are added to root which is not ideal. So we should make our own directory. 

Add a `WORKDIR` entry. 

When you `sh` into the container, you will go directory to that directory. Not `/`. 


# Unnecessary Rebuilds
If modifying the source code, after the container is already running, nothing will happen or change. That's because we made a snapshot of the old version. There is additional work that is needed there. 

Would need to rebuild the container. 

## Minimizing Cache Busting and Rebuilds
You can add multiple `COPY` keys to a Dockerfile. This can be used to help reduce rebuild times and utilise caching more. This way you could prevent the need to download libraries again, again and again when small changes are made which did not require a package to be re-downloaded. 

**Example**
```Dockerfile
COPY ./package.json ./

RUN npm install

COPY ./ ./
```

Copy just the dependancies, then install NPM (if already done, uses the cache). 

Then COPY the rest of the files. If changes are required there, it won't take too long to build the full image because we do not need to rerun `npm install`. 
# Application 2 / Project 2
A web app that shows how many times a person has visited that web app. 

Needs 2 components. 

So a node app and a redis server. 

Don't want to put both Node App and Redis on the same container because we will have scaling issues such as different values for the same variable stored in Redis - as each container will have it's own value. 

So we will need to have 1 redis instance and multiple docker node containers which can scale. 

Found here: `/Users/daniel.homan/Desktop/Docker/visits`

This will also use docker-compose

# Docker Compose
Different to the docker CLI in that it basically will allow you to run certain docker tasks easily without having to constantly retype them again and again. Can easily set up multiple containers at the same time and have them linked together. 

We are going to take the commands we have been typically running and add them to a `docker-compose.yml` file which we will run using the docker-compose CLI. 

## Docker Compose Commands
`docker run myimage` -> `docker-compose up`

`docker build .`         ->
										     -> `docker-compose up --build`
`docker run myimage` ->

## Stopping Docker Compose Containers
`docker run -d redis` will create a redis container in the background and not provide any output. 

To stop that container. 
`docker stop <id>`

Can stop multiple containers at the same time. 

`docker-compose up -d`

`docker-compose down`

# Handling Crashes with Applications 
If we look at the node app `index.js` we can see we added an `exit(0)` to make sure the app crashes. 

After encountering this, if we run `docker ps`, we will only see the redis instance running. 

So now we need to handle automatic container restarts. 

# Automatic Container Restarts 
Restart policies can be added to containers. 

By default, "no" restart is added to a container. So if it ever stops or crashes it never spins back up. 

You also have `always` -> if stopped for any reason, it will come back up. 
`on-failure` -> only restart if the container stops with an error code
`unless-stopped` -> always restart unless we (the developers) forcibly stop it. 

If specifically adding "no", it must be added in quotes like the below: 

```yml
version: '3'
services: 
  redis-server:
    image: 'redis'
  node-app:
    restart: 'no'
    build: .
    ports:
      - "4081:8081"
```

# Container Status with Docker Compose
`docker-compose ps`

similar to `docker ps`

Need to run this from the directory where the docker-compose.yml file is. 

# Development Workflow
Development -> Testing -> Deployment

Do everything through GitHub.

Write tests through TravisCI to make sure things are working correctly. Assuming TravisCI pulls the data perfectly, TravisCI will run the tests and if all works well, it will push everything to AWS. 

Recommend having two Dockerfiles -> 1 for development (`start`), 1 for production(`build`).

## NPM Stuff
`npm run start` -> starts up a development server - this is for development only, not prod, etc.

`npm run test` -> Runs tests assoaciated with a project

`npm run build` -> builds a production version of the application

## Development Dockerfile 
Has the name `Dockerfile.dev`. Production one is simply `Dockerfile`.

Dir:` /Users/daniel.homan/Desktop/Docker/node-app`

To build the dev file, we do: 

`docker build -f Dockerfile.dev .`

`-f` allows us to use a file for the build, so simply pass the file name. 

Dependancies can be duplicated when running the build command so just remove the node modules folder and run the build command again - this will build the file very quickly and there will be no duplicates. 

## Starting the Container
`docker run <container_id>`. Always be sure when something is exposed over a port to add the port or `-p` tag. 

# Docker Volumes
Volumes allow you to mount files which can be accessed by the container. Good way to prevent stopping containers and rebuilding when making tweaks to applications or UI tools. 

Can think of it as a mapping of a file inside a container to a file outside of the container. It's essentially a reference point. 

The command to do so would look similar to the following: 

`docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_id>`

The second `-v` gets the current working directly and maps it to `/app` running in the container. 

The first `-v` puts a bookmark on the folder. 

## Bookmarking Volumes


# Shorthand Docker-Compose
Docker compose is used to help shorten queries. 

Add a volumes section into the docker compose yml. 

Everything is in the `frontend` folder. 

# Running Tests
When it comes down to running tests, we can use `npm run test` to do so for the app that is present but what happens if we have the container running and we want to add a new test to which the container was started with `docker build -f Dockerfile.dev`? If this is the case, then nothing would happen because there are no volumes present. So we have 2 options, we can keep the container running and start up a docker-compose command which will have volumes established and we exec into the container, we go into the files and update accordingly (not the best way of doing this), or we set up different services in the `docker-compose.yml`. 

One service for the app, another for the tests. This is also not the perfect solution. 

# Docker Compose for Running Tests
Just build an additional service for the mapping and load everything into it. Just exec into one and make changes as needs be. 

# Multi-step builds

Can have 2 base images instead of just one. Lets say I want to use node because of my dependancies for an app but I also want to use Nginx, it would mean I basically need an image that can handle both node and also nginx but what we can do is introduce 2 phases: Build and Run phase. 

Build will use the node:alpine image, copy the files, install dependancies and start a production service. 

Run will be used to start using nginx (during this phase we can specify the use of another base image), we copy the result of the npm run build command and then start nginx. 
