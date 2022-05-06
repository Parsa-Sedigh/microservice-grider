# 03 - Running Services with Docker

## 53-001 Deployment Issues:
How we can deploy this app?
We can go out to digital ocean, AWS or ... and rent a virtual machine. But it could get more challenging.
Let's imagine that our comments service is starting to become overburdened. Maybe we have a tremendous number of users coming into our app and trying to create
comments and so in future, we might decide that we need to create a second or third instance of this comments service just to handle this additional demand.
An easy way to do this would be to say on the same virtual machine, let's create two new copies of that comments service and for example those copies run on port
4006 and 4007. Whenever someone tries to create a comment, we can then load balance between these three different comments services.
Load balance means just randomise which server the incoming request goes to?

There are a couple of challenges with this approach. 
1) These additional copies of comments service would have to be allocated some new port on this machine. That's relevant, because remember our event-bus 
needs to know the exact IP address(domains?) and port of all the different services that it's going to send events to.
So in other words, as soon as we create these additional copies of the comments service, we would have to open up our event-bus code!
and we would have to add in some additional lines to send the events to those new copies.
**Important**: So if we follow this approach, then we're gonna start to couple or very directly tie the number of services that we are running with the
actual implementation of our code(event-bus code). If we ever decide to increase or decrease the number of comments service servers running at any given time,
we're going to have to make a change to our event-bus code and deploy that change as well. 

This whole scenario gets even more complicated if we see that the comments services that we added on earlier as copies, are overburdening the ONE virtual machine
that we currently have. So let's imagine that we decide to get a second VM and run those two additional comments services on the second machine.
Maybe they are still listed at port 4006 and 4007, but now the event-bus not only needs to know or keep track of ports 4006 and 4007, but it also is gonna have to
have some code to figure out how to reach out to this other VM and eventually get access or send some events to the comments services there.
So one again we would have to open up our event bus file and rather than make your req to localhost, we have to put in the ip address of that other VM.
Another scenario: Let's imagine that our web site is very popular at just certain times of the day, so let's say at 10 am a lot of people use the comments service,
so at 10am in morning we would want to have those additional copies of the comments service running. But maybe at 1.00 am in morning, nobody is coming to our app, 
so in order to save money on our hosting fees, we decide at 1:00 am to temporarily shut down the second VM that is running two copies of comments service.
Now in this scenario, again, the event-bus needs to know that that second VM is now dead and it should not attempt to send any events over there.
To handle that, in event-bus we need to add some code and say: if it is not 1:00 am and put the codes for sending events to /events of that additional VM services,
inside that if statement.
Currently, we're trying to put together this event-bus and have it communicate between all these different services in a very direct and imperative fashion.

So currently event-bus also needs to keep track whether or not the services are running currently.

**Note**: We need sth that's gonna keep track of all the different services that are running inside of our app, has the ability to maybe create new copies of a service
on the fly and make sure that we have sth as well that can automatically figure out whether or not a service is running and whether or not we should maybe try to
make contact between two given services. 
We need docker and kubernetes.

## 54-002 Why Docker:
A docker container is like an isolated computing environment. It contains everything that is required to run one single program.
We're gonna end up creating separate docker containers to run each of our individual services.
If we ever need multiple copies of some service, say if we need multiple copies of comments, then we will start up a second docker container.
So there's gonna be a one to one pairing between docker containers and each instance of our running program.

Why docker?
- Currently, running our app makes big assumptions about our environment. Currently, to run our app, we run npm start. Well to run npm itself,
  we're assuming that it is installed on our local machine. We're also making the assumption that node is installed on our local machine as well.
  So just to run our program, we're making those really big assumptions.
- Running our app requires precise knowledge of how to start it(npm start).

Recap: Not only starting up our app is potentially confusing and kind of mysterious, but we also have these kind of implicit requirements about the 
environment that we're running our program on.

By creating containers, we're gonna warp up all the dependencies that a program requires. That means we're gonna wrap up npm and node into this little isolated
computing environment and we're also gonna include in that container, some information about how to start up and run our program as well.

Docker makes it easy to run any program.

## 55-003 Why Kubernetes:
Docker makes it easy to start up and run our different programs.
Kubernetes is a tool for running a bunch of different containers together. When we're using it, we give it some config files, these config files are gonna 
tell kubernetes about the different containers that we want to run in our app. Kubernetes is then going to create these containers that are going to run our 
programs for us and it's gonna handle communication or essentially network reqs between all these different containers as well.
So we can imagine kubernetes as a tool to run some different programs and make communication between those programs easy and straightforward.

We give kubernetes some config to describe how we want our containers to run and interact with each other.

A cluster in kubernetes is a set of different virtual machines. It can have as few as one VM or thousands VMs. We refer all these VMs as nodes. They are 
all managed by sth called a master. Master is a program that's going to manage everything inside of our cluster. All the different programs that are running, all
the different aspects of these VMs and many other things.

We're gonna tell kubernetes to run some programs for us, when we do so, it's gonna take our program and then more or less randomly assign it to be executed by one 
of these nodes(a node is just a VM).

We might write a config file that says: run two copies of our posts service and also make sure that we can easily access these posts services after they have been 
created, so please allow copies of posts to be accessible from network.
We write the configuration file and then feed it into that master thing. Master is gonna read those instructions and try to implement all the steps we've 
written inside those config files.
In this case, it's gonna attempt to create two different copies of our posts service wrapped up inside of containers.
These will be randomly assigned, more or less randomly, there actually is some science to it, but we can imagine that they will be assigned to be executed
by some different VMs or nodes inside of our cluster.

Now, in this scenario we're kind of back in the same problem we had before where it could potentially be hard for sth like our event-bus to figure out in how
to reach out to one of the copies of posts service and the other one.
The big thing that kubernetes does for us, is give us the ability to just kinda arbitrary send reqs or communicate between these different services with some kind of 
third party thing that we're gonna create. 
So, you can kinda imagine down here that we're gonna create sth inside of our cluster that we can just send req to and it will automatically figure out
how to route reqs off to the appropriate service that is running inside of our app.

Hey! We said we didn't want our services to communicate directly with each other? 
Well, yes. Remember we're not using sync communication between our services, but there still are programs that ultimately do have to talk with each other
inside of our microservices apps.

## 56-004 Don't Know Docker_ Watch This:

## 57-005 Note About Docker Build Output and Buildkit:

## 58-006 Dockerizing the Posts Service:
In order to dockerize the posts service, we're gonna have to create an image out of it.

To make an image, we make a dockerfile and then run the docker build command.

In dockerfile, first we need to specify our base image, then we specify a working directory which is going to set the working directory for the REST of
all the commands inside of our Dockerfile.
The Dockerfile would be:
```dockerfile
# specify the base imgae
FROM node:alpine

# set the working directory to '/app' in the container. All FOLLOWING commands will be issued relative to this directory
WORKDIR /app

# copy over only the package.json
COPY package.json ./

# install all deps
RUN npm install

# copy over all of our remaining source code
COPY ./ ./

# set the default command to run when the container starts up
CMD ['npm', 'start']
```
we usually don't want to include any installed deps that are on our local machine into the resulting image. 

In order to make sure that we don't send all those deps over, during the image building process, we're gonna create the .dockerignore .
That file is gonna make sure that during the image building process, we don't attempt to copy over that node_modules directory to the image.
Because the image is gonna install all the relevant deps when it is being BUILT on it's own.

Now run docker build . inside the posts directory.

At the end of the logged build command, we get the id of the image that was just built. So we can now create an instance of that image or 
in other words, a container, by copying that id and running: docker run <id of image> .

## 59-007 Review Some Basic Commands:
In docker build command, whenever a new image, we can add a -t flag which will tag the image that is created from the build command.
If we add the -t flag , then we can easily create a new container out of that image, using EITHER the image ir or the image tag which was 
specified by the -t flag when building the image.
```shell
# build an image based on the dockerfile in the current directory. Tag it as 'parsa/posts'(this name usually is: '<docker id>/<name of the project>').
docker build -t parsa/posts .

# create and start a container based on the provided image id or tag
docker run [image id or image tag(in this case is: parsa/posts)]
```

Whenever we start up a container, we can either just use docker run by itself, or alternatively we can override the default command of the 
container by providing both -i and -t flag and then type in the command that we want to execute instead of the default one of the container.
For this, you can run:
```shell
docker run -it [image id or image tag] [the command to be used instead of the default one of the container]
```
So for example, we could start up a container and startup a shell inside there instead of actually running the default command(which in package.json
is npm start). For this, we can run:
```shell
docker run -it <name of image> sh
```

We can use this command to print out information about all of the running containers:
```shell
docker ps
```

Once a container is running, we can execute arbitrary commands inside of it:
```shell
docker exec -it [container id] [arbitrary command that we want to run inside of container]
```

This will print out all the logs that have been emitted by the primary process INSIDE of that container.
```shell
docker logs <container id>
```

For example, if you run a container that is a nodejs project and has a default command of npm start, it would emit some logs over STDOUT.
Now in a second terminal window, we can run: 
```shell
docker ps # the running container id that you want
docker logs <the id that you got>
```

## 60-008 Important Note Regarding Node v17:

## 61-009 Dockering Other Services:
Now we want to use kubernetes to run all of the containers and get all those containers communicating together easily.