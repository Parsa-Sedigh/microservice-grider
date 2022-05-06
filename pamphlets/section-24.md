# 24 - [Appendix A] - Basics of Docker

## ??-001 Why Use Docker:
Why use docker?
Docker makes it easy to install and run software without worrying about setup or dependencies. 

## ??-002 What is Docker:
Docker ecosystem:
- docker client(docker CLI)
- docker server
- docker machine 
- docker images
- docker hub
- docker compose

Docker is a platform or ecosystem around creating and running containers.

An image is a single file with all the deps and config required to run a program(like redis). Image is a single file that gets stored on your hard drive and 
at some point in time, you can use this image to create sth called a container.
A container is an instance of an image and you can kind of think of it as being like a running program. A container is a program with it's own
isolated set of hardware resources. So it kinda has it's own set or it's own little space of memory, has it's own space of 
networking technology and it's own little space of hard drive space.

## ??-003 Docker for Mac _ Windows:
We interact with docker cli a lot. The docker client itself doesn't actually do anything with containers or images. Instead, the docker client is just a 
tool or a portal of sorts to help us interact with another piece of software that is included in the docker for windows or mac package and that piece of 
software is called the docker server or docker deamon. This program is the actual tool or piece of software that is responsible for creating
containers, images, maintaining containers, uploading images and ... .
So we interact with docker client(we issue commands to it) and behind the scenes, this client is interacting with the docker server.
We're never gonna reach directly to docker server, it's kina running behind the scenes.

## ??-004 Installing Docker on macOS:
Login to docker to startup docker. Now we need to verify docker installation. To do this, run: docker version.

## ??-005 Installing Docker for Windows Home users:
## ??-006 Installing Docker for Windows Professional:
## ??-007 More Windows Professional Setup:
## ??-008 One Last Piece of Windows Professional Setup:
## ??-009 Installing Docker on Linux:
## ??-010 Using the Docker Client:
run: 
``` shell
docker run hello-world
```

docker run hello-world will start up the docker client or the docker CLI. Again, the docker cli is in charge of taking commands from you, kinda doing a little
bit of processing on them and the communicating the commands over to sth called the docker server and it's that docker server that is really in charge of the 
heavy lifting when we ran the command.
When we run:
``` shell
docker run hello-world
```
that meant that we want to start up a new container using the image with the name of `hello-world`.
The hello-world image has a tiny program whose sole purpose, is to print out the hello-world message.
Now after the command arrived at docker server, a series of actions occurred in background, docker server first will check to see if it already had a 
local copy on your machine of the hello-world image or file. So it would look into sth called image cache(downloaded images are there). Because it was empty,
it would reach to a free service called docker hub. After downloading the image from there, it will store that image in image cache. Then it will use it 
to create an instance of the image.

## ??-011 But Really... What's a Container:
How OS runs on our computer?
Most operating systems has sth called a kernel. This kernel is a running software process that governs access between all the programs that are running on 
your computer and al the physical hardware that is connected to your computer as well.

If you write a file to hard drive, it's technically not nodejs that is speaking directly to the physical device. Instead, nodejs says to your kernel:
hey, I want to write a file to the hard drive(physical device). The kernel then takes that information and eventually persist it to the hard disk.
So the kernel is always kind of this intermediate layer that governs access between these programs and your actual hard drive.
These running programs like terminal or nodejs or spotify or ... , interact with the kernel through things called system calls. These are essentially like
function invocations. The kernel exposes different endpoints to say: hey, if you want to write a file to the hard drive, call this endpoint or this function 
right here. It takes some info and then that info will be eventually written to the hard disk or memory or ... .

Imagine:
We have two programs running on program, program A wants python v2 and program B wants python v3 and we only have python v2 installed on our machine and 
for whatever reason we can't install another version of python. What we could do?
We can make use of a operating system feature known as name spacing. With name spacing, we can look at all of the different hardware resources connected to our 
computer and we can segment our portions of those resources. So we could create a segment of our hard disk specifically dedicated to housing python v2 and we could 
make a second segment specifically dedicated to housing python v3. Then to make sure that program A has access to the segment that has the python v2 and program B
has access to segment of hard disk that has python v3, anytime that either of them issues a system call to read information off the hard drive, the kernel
will look at that incoming system call and try to figure out which process it(the system call) is coming from?
So the kernel could say: Ok, if program A is trying to read some info off the hard drive, I'm gonna direct that call over to that little segment of hard disk that has
python v2 and anytime program B makes a system call to read the hard drive, the kernel can redirect that over to the segment of hard disk that has python v3.

So by making use of this kind of namespacing or segmenting feature, we can have the ability to make sure that program A and B are able to work on the same machine although
they need different versions of python.
SO this entire process of kind of segmenting a hardware resource based on the process(like program A) that is asking for it, is known as namespacing.
With namespacing we're allowed to isolate resources per a process or a group of processes and we're saying that anytime this particular process asks for a 
resource, we're ognna direct it to this one little specific area of the given place of hardware.

Now namespacing is not only used for hardware, it can be also used for software elements. For example, we can namespace a process to restrict the area
of a hard drive that is available or the network devices that are available, or the ability to talk to other processes or the ability to see other
processes. These are all things that we can use namespaces for to essentially limit the resources. We're kinda redirect request for resources from a particular
process.

Very closely related to this idea of namespacing, is another feature called control groups(cgroups). A control group can be used to limit the amount of resource
that a particular process can use. So namespacing is for saying: "hey, this area of the hard drive is for this process." A control group can be used to limit the 
AMOUNT of memory that a process can use, the amount of cpu, the amount of hard drive, input output and the amount of network bandwidth as well.
So these two features(namespacing and control groups) can be used to kinda isolate a single process and limit the amount of resources it can talk to and the amount
of bandwidth that it can make use of.

The entire of running process plus the little segment of a resource that it can talk to is what we refer to as a container.
**Learn: A container is a process or a set of processes that have a grouping of resources specifically assigned to it.
In a container, a process(or multiple processes) sends a system call to a kernel, the kernel is gonna look at that incoming system call and direct it to a very
specific portion of the hard drive, the RAM, CPU or whatever else it might need, so portions of each of these resources or hardware are available to process.**

Anytime we talk about an image, we're really talking about a filesystem snapshot. So an image is kind of like a copy paste of a very specific set of directories or 
files. So we might have an image that contains just chrome and python as it's FS(filesystem) snapshot. An image will also contain a specific start-up command.
So here's what happens behind the scenes when we take an image and turn it into a container:
First off, the kernel is gonna isolate a little section of the hard drive and make it available to just this container. So we can kinda imagine that 
after that little subset of hard-drive is created, the file snapshot inside the image is taken and placed into that little segment of the hard drive and 
so inside of that very specific grouping of resources, we've got a little section of the hard that has just chrome and python installed and essentially nothing else.
The startup command is then executed, which we can kinda imagine, in this case is like startup chrome and so chrome is invoked and we create a new instance of that 
process and that created process is then isolated to this set of resources(RAM, network, CPU) inside the container.

That's the relationship between a container and an image and how an image is eventually taken and turn into a running container.

## ??-012 How's Docker Running on Your Computer:
A container is a running process, along with a subset of physical resources on your computer that are allocated to that process SPECIFICALLY. 
An image is really a kind of a snapshot of the filesystem, along with a very specific start-up command as well.

For separating these resources that are used by containers, we use a technique called namespacing.
We could limit the AMOUNT of resources used by these, using control groups.

Now this feature of namespacing and control groups is not included by default with all operating systems. These features are specific to linux OS.
So how are we running docker(currently I'm running docker client and docker containers) right now on mac?
When you install docker for windows or docker for mac, you installed a linux virtual machine. So as long as docker is running on top of that menu bar in mac,
you technically have a linux VM running on your computer. Inside this virtual machine is where all these containers are gonna be created.
So inside the virtual machine, we have a linux kernel and that linux kernel is gonna be hosting running processes inside of containers and it's that linux kernel 
that's going to be in charge of limiting access or kinda constraining access or isolating access to different hardware resources on your computer.
You can see this linux VM in practice by running docker version and look at Server section and OS/Arch and you'll notice that it probably 
doesn't have your OS listed. For example it could be linux/amd64 . So it's kinda specifying we're running a linux VM and that's what's being used 
to host all thse different containers that we're working with.

## ??-013 Docker Run in Detail:
Creating and running a container from an image:
```shell
docker run <image name>
```
When we run this command, chances are that somewhere on our hard disk, is an image that has that filesystem snapshot with one single program inside of it.
By running this command, we took that little snapshot of the hard drive, we stuck it into that little container thing(or this kind of grouping of resources) and 
then we executed the command `run hello world`. So the running process in the container is hello world. In this case, that process ran and very quickly exited
because it was just a console log.
Let's look at some of the features around `run` command.

## ??-014 Overriding Default Commands:
Remember anytime that we execute `docker run` with an image, we not only get that filesystem snapshot, but we also get the default command that is kinda 
inside the image and is supposed to be executed after the container is created. 
A variation on docker run command that we're gonna look at, is gonna give you a way to override that default startup command of image that is gonna run after the
container of image is created:
```shell
docker run <image name> <command>
```
The command in above is the alternate command to be executed inside the container after it starts up(after the container is created from the image). 
This is an override. So whatever default command is included inside of the image, is not going to be executed.
EX)
```shell
docker run busybox echo hi there
```
If you run:
```shell
docker run busybox ls
```
it will print out all the folders that are not belonging on your computer. Those printed folders are folders that exist solely inside that container.
Busybox image has some default filesystem snapshot which are: bin, dev, etc, home, proc, root and ..., and some presumbaly default command.
So when we create a new container out of that image, we take this filesystem snapshot, we stick it in as the folder as the hard drive segment for that
program of container and then the command that we execute in this case was ls.
Now run:
```shell
docker run hello-world ls
```
We get an error. Let's run: docker run hello-world echo hi there and we get a similar error message. Why?
When we run the alternate commands like echo ... and ls commands with busybox image, those commands work because ls and echo are two programs that exist
inside of the busybox filesystem image. Somewhere inside of this folder system of busybox, is a command or an actual ls executable and a echo executable as well.
So we can safely execute those commands with busybox because those are programs that exist inside that filesystem of busybox image.
However with hello-world program, the only thing that exists inside that filesystem snapshot is a single program, just one single file and all that thing does is 
to print hello world.
So these startup commands that we're executing(whether default start up command or the alternate that we write) are being based upon the filesystem included
with the image and if we try to execute a command inside the container that uses a program that doesn't or is not contained within the filesystem of image,
we're gonna see that error.

## ??-015 Listing Running Containers:
```shell
docker ps
```
This command will list all the different running containers that are currently on your machine.
Till now, we've only been running images or creating containers that run very quickly and then immediately closed down. So if we want docker ps
to be meaningful at all, we have to have some container that is running for some longer amount of time.
In order to get a container running a little bit longer, we could substitue the command that is executed when that container starts up(so we use the alternate
command with docker run), so rather than running docker run busybox hello there, for example run: docker run busybox ping google.com .
ping google.com is gonna attempt to ping google servers and measure the amount of latency.
Now run docker ps in a second window of terminal.

A random name will be generated for each running container.

We can modify the docker ps command a little to show all containers that have ever been created on our machine. Run:
```shell
docker ps --all
```
The containers will shut down on our behalf or shut down naturally.

## ??-016 Container Lifecycle:
docker ps --all will show all the containers we'd ever started up on our machine.

To start up a new container from an image, we use the docker run <image name> command.
This command creates AND runs a container. So creating a container and starting it up, are two separate processes. So:
```shell
docker run <image name> = docker create <image name> + docker start <container id>
```
docker create command creates a container out of an image and docker start, starts a container.

**Learn**: The process of creating a container is where we kind of take the filesystem in the image and kind of prep it for use in the new container that is gonna
get created out of that image. 
When we create the container, we're just talking about kind of prepping or setting up that filesystem snapshot of image, to be used to create the container.
To actually start the container, that's when we execute that startup command of image that might startup the process of like hello world or in the case of busybox,
one that we used the echo hi there or whatever process is supposed to be executed inside the container.

So creating a container is about the filesystem. Starting it, is about actually executing the start up command.

After running docker create(which prepares the container by getting that filesystem ready) you get the id of the container that was just CREATED. 
Now we can execute the startup command inside of that created container by running: docker start -a <id of container>
If you don't use -a argument in docker start <id of container>, we just get the id of container again!
**Learn**: The -a command is what's gonna make docker actually watch for output from the container and print it out to your terminal.
So the -a argument means: hey, kind of attached to the container so to speak, watch for the output coming from it and print it out at my terminal.

So there's a difference between docker run and docker start by default. The docker run BY DEFAULT is going to show you all the logs or all the information
coming out of the container, but by default, docker start is the opposite, docker start is NOT going to show you information coming out of the terminal.

## ??-017 Restarting Stopped Containers:
We can easily stop and then start containers again at some point in the future.

To start a container back up, we can take it's id using docker ps --all and then execute docker start -a <id of container>.

First off, we ran docker create or docker run , that took the filesystem snapshot of the image and essentially got a reference to it inside the container.
We then provided that override command of echo hi there. So that was the primary running command inside the container. That command ran, then completed, the
container exited naturally and we then ran the docker start again a second time, what happened, was the running process or the primary command was issued a 
second time inside the container.

When you have a container that's already been created(even if it's currently exited), we cannot replace that default command.
As soon as you start it up with the deafult command, that's it!
So we can NOT do sth like: docker start -a <id of exited container> <an alternate command for that exited container>.

So once we have started up a container and let it exit, we can start it back up again, which is gonna reissue the default command that was used
when the container was first created(but we can't change that default command).


## ??-018 Removing Stopped Containers:
If you run docker ps --all it would show the stopped containers. So those containers are essentially just taking up space on our computer.
So let's entirely delete them and not just leav them in this stopped state. 
To delete all of the stopped containers(and some other stuff), run: 
```shell
docker system prune
```
This command is not only going to delete stopped containers, it's also going to delete some other stuff as well. Most notably your build cache.
The build cache is any image that you have fetched from docker hub. So after running this command, you have to redownload images from docker-hub.

## ??-019 Retrieving Output Logs:
If you forgot to use -a with docker run in order to see the output from that started container, you can use:
```shell
docker logs <the id of the container that we want to get output from>
```
The `logs` command is used to look at a container and retreive all the information that has been emitted from it.
EX)
docker create busybox echo hi there
this would return an id: <id>
Now run: docker start <id>
OOPS, we forgot to add the -a flag. No problem, use:
docker logs <id>
which would show hi there log from that stopped container.

By running docker logs, we're not rerunning or restarting the container in any way. We're just getting a record of all the logs that have been emitted 
from that container.

This command is good for inspecting a container for debugging and ... .

## ??-020 Stopping Containers:
To stop a container you can use either:
- stop a container: docker stop <container id>
- kill a container: docker kill <container id>
What's the different between them?
Here's what happens behind the scenes:
When you issue a docker stop command, a hardware signal is sent to the primary process inside that container which you want to stop.
In case of docker stop command, we send a sigterm message which is short for terminate signal. It's a message that's going to be 
received by the process, telling it essentially to shut down on it's own time.
Learn: Sigterm is used anytime that you want to stop a process inside of your container and shut down the container and you want to give that process inside 
there a little time to shut itself down and do a little bit of cleanup. 
A lot of languages have the ability for you to listen for these signals inside of you code base and as soon as you get that signal, you could attempt
to do a little bit of cleanup or maybe save some file or emit some message or sth like that.

On the other hand, the docker kill command issues a sigkill or a kill signal to the primary running process inside the container.
Sigkill means: you have to shut down right now and you do not get to do any additional work.
Ideally we always stop a container with the docker stop command in order to give the running process inside of it a little bit of time to shut down itself.
Otherwise, if it feels like the container has locked up and it's not responding to the docker stop command, then we could issue docker kill command instead.

When you issue docker stop to a container, if the container does not automatically stop in 10 seconds, then docker is gonna automatically fall back to
issuing the docker kill command.

When you run docker stop <id of container> and the container is a container that is running an image like busybox but with alternate command of ping <some server>,
it would wait 10 seconds and then gets shut down.
Why? Because it turns out that the ping command doesn't properly respond to a sigterm message. The ping command doesn't really have the 
ability to say: Oh yeah I understand you want to me shut down. The ping command just wants to run forever, so it needs a kill signal which is gonna be issued
after 10 seconds in case of docker stop command.

Now for running the container again:
```shell
docker start <id of stopped container>
docker run <id of container>
```
It's good to use docker stop.

## ??-021 Multi-Command Containers:
If you have installed redis but without docker involved, you can run this command to start an instance of the redis server:
redis-server
Now again, I'm able to run this command because I installed redis on my local machine without docker involved.
Now that you have started the redis-server, you can use redis-client and the entire purpose of redis-client is to reach into that
redis-server and poke around inside of it.
So open a second terminal window besides the window for running redis-server and run:
redis-cli
and it gives you a prompt to issue commands directly to the running redis-server.

Let's startup redis using docker. So stop redis-server and redis-client and run:
```shell
docker run redis
```

Now if you run redis-cli in a second terminal window, you will get: could not connect to redis. Why?
We started up a new container that is running redis-server as it's running process. Now redis is running ONLY inside this container.
So if I try to run redis-cli outside the container, outside the container I have NO access to anything that's going on inside
there and so there is no redis-cli command to run outside the container.
If we want to start up redis-cli, we need to somehow get INTO that container and execute a second command inside of it.
So we need to startup a SECOND program or running process inside the container.
So inside of that container, we need to have 2 programs or running processed running TOGETHER.

## ??-022 Executing Commands in <<<<<RUNNING>>>>>> Containers:
We need to start redis-cli besides redis-server inside of that container. To do so, we use:
```shell
docker exec -it <id of the container> <command to be executed INSIDE the container>
```
With this command we execute additional command in a container. 
In this command:
docker: reference the docker client
exec: run ANOTHER command
-it: allow us to provide or type input directly to the container

So for our example, run: 
docker ps to get the running redis-server container id. Then run:
docker exec -it <id of container that you got> redis-cli
After doing that, you see we get that command prompt to interact with that redis-server inside that container.

So by using the `exec` command, we were able to start up a second running program(or execute other commands that would done immediately and not 
start a running process) inside of our container.
The -it flag allowed us to enter in text on our keyboard and have it be sent into that running container.

If you don't use -it flag in order to run a command inside of a container, the container would run but we won't get any possibility of 
writing text input to that command.
So in our case, we would get kicked back into the original prompt and not the redis-cli prompt, because redis-cli was started up,
but it very quickly realized: "hey, I don't have any possibility of getting any text input". It decided to just entirely close down
and kick us back to our terminal.

But what that -it flag really means?

## ??-023 The Purpose of the 'it' Flag:
What does -it is doing for us?
The first thing we need to understand is how processes run inside of a linux environment?
**Learn:** When you're running docker on your machine, every single container that you are running, is running inside of a virtual machine which is running linux. So these processes(container processes) are really being executed inside of a linux world, even if you're on mac or windows.

Let's imagine we have a container with 3 different running processes, so all inside in theory of a running container, or really inside of a 
linux environment. 
Those processes inside that container are: 
- ping google.com
- echo hi there
- redis-cli

Every process that we create in a linux environment has 3 communication channels attached to that process. We refer to them as:
- STDIN
- STDOUT
- STDERR

These channels are used to communicate information either INTO the process or OUT of the process.

- STDIN is used to communicate information INTO the process. So when you're at your terminal and you type stuff in, the stuff you type is being
  directed into a running standard in(STDIN) channel attached to that process, for example redis-cli .

- The STDOUT channel that is attached to any given process, is going to convey information that is coming from the process. 
  So standard out(STDOUT) might be redirected over to your running terminal and that's gonna end up as being stuff that is going to show up on the 
  screen.

- STDERR is very similar, but it conveys information out of the process that is kind of like an error in nature.
  So if the process has some error inside of it, that's gonna be communicated to the outside world over the STDERR channel and very similar to 
  STDOUT, that's going to be redirected to show up on the screen of your terminal.

Now how does these stuff relate to -it flag when we do docker exec -it?
-it is actually: -i -t 
-i means when we execute this new command inside the container, we want to attach our terminal to the STDIN channel of that new running
process(like redis-cli). So by adding the -i flag, we're saying: make sure that any stuff that I type, gets directed to STDIN of that process.
The -t flag is what kind of makes all this text show up a little bit pretty. In reality, it's doing a bbit more than that. But at the end of the 
day, the real effect that the -t flag, is to make sure that all the text that you are entering in and that is coming out, shows up in a nicely
formatted manner on your screen).
If you don't use -t flag, you don't get the nice indentation and auto-completion.

So -it allows us to have stuff that we type into our terminal, directed into that running process and it allows us to get information out, 
back to our terminal.

## ??-024 Getting a Command Prompt in a Container:
One last use of docker exec command is to get shell access or terminal access yo your running container.

A very common thing that you'r e gonna want to do when using docker, is to get shell access or terminal access to your running container.
In other words, you're gonna want to run commands inside of your container without having to rerun 
docker exec -it <container id> <command to run in container>, docker exec ... and ... again and again.

So we want to open up a shell or terminal in the context of your running container. For this, run:
```shell
docker exec -it <container id> sh
```
This command is good for debugging the container.

If you're in a container and it feels like you can't hit ctrl+c to exit, you can always ctrl+d to exit from the shell in that container or 
type `exit` and enter.

Learn: sh is the name of a program. and when we use it with docker exec, it's a program that would get executed inside of that container.
sh is a command processor or a shell. It's sth that allows us to type commands in and have them be executed inside that container.

If you're on mac, you're possibly using bash, on windows, powershell or gitbash.
These are all programs that allow you to type commands into your terminal and have them be executed and so when we start up `sh` inside of 
our container, that's just another command shell that we can use to execute commands inside container.

Traditionally, a lot of containers that you're going to be working with, are probably going to have the sh program already included.
Some more complete versions of containers or images are going to also have the bash command processor as well. So in some cases, you can make 
use of bash directly, in vast majority, you're probably going to be able to run the shell inside there to a startup a command prompt and type in 
some commands by using sh.

command processors:
- bash
- powershell
- zsh
- sh

So we're gonna use docker exec -it <container id> <command> with sh as the command so:
docker exec -it <command> sh

## ??-025 Starting with a Shell:
Instead of using docker exec for using sh inside a container, we could also run docker run along with that -it flag and startup a shell
immediately when a container FIRST starts up.

Now of course, if we start up a shell right when the container first starts, that's going to displace or keep any other typical or default command
of the image from running. But sometimes it is useful to get a empty container(because the primary program that's gonna run inside the container,
in this case is sh) with a shell inside of it and just be able to poke around and not have any other process running. So run:
```shell
docker run -it busybox sh
```
We used -it so that we can attach to STDIN on the process that we're about to start up.
In this command, the program or the primary command that we're going to execute inside the container, will be sh. So this means, start up a 
new container out of the busybox image, run the sh program which is a shell and attach to STDIN of that program(sh).
Anytime you want to poke around a container, you can use this command.

Note: The downside to use docker run with -it flag and sh as primary process(which in this case would be the only process that is running inside
the container, because we just started up the container from image), is that chances are you're not going to be running any other process.
It's a little bit more common that you're going to want to start up your container with a primary process of like maybe your web server or 
whatever it might be and THEN, attached to it, a running shell by using the docker exec command.
But anyway, you're now aware of this that you can use docker run with sh as it's primary process.

## ??-026 Container Isolation:
In the last video, we learned how to start up a shell inside of a container the instant that it was started.

Between two containers, they do not automatically share their filesystem.
To confirm this, run:
docker run -it busybox sh
now you can run:
ls

Now run a second terminal window and start a second container with the same parameters, so run the exact command.

Now run the third window and run: docker ps

If in the first terminal window, create a new file. Now in second terminal window, we can't see that created file which you created in first
container.
So these two running containers have absolutely completely separate filesystems and there's no sharing of data between the two.
So in general, unless we specifically form up a connection between two containers, we really consider them to be more or less completely isolated from
each other and totally separated.

## ??-027 Creating Docker Images:
Let's build our own custom images. To create our own image:
create a file called dockerfile which is a plain text file that is going to have a couple of lines of configuration inside of it.
**Learn:** This configuration defines how our container behaves or more specifically, what different programs it's going to contain and 
what it does when it starts up as a container.

Once we create that dockerfile, will then pass it off to the docker-client which is the docker cli that we use at our terminal.
In turn, the docker client will provide the file to the docker-server, the docker-server is what is doign the heavy lifting for us.
It's going to take the docker file look at the configurations and then build a usable image that can then be used to start up a new container.

### creating a dockerfile:
- Inside every dockerfile, we're gonna specify a base image.
- add in some additional configuration to run commands, to add in some deps or some more software, that we need, to successfully create and execute our
  container.
- specify a startup command to run on container startup. So anytime we take that image and create a container out of it, it will be this command that
  is executed to essentially boot up or start the container(however, if you provide an alternate command with docker run, that command will executed instead of the specified image startup command). 

## ??-028 Buildkit for Docker Desktop v2.4.0+ and Edge:

## ??-029 Building a Dockerfile:
We're gonna create a dockerfile that is going to create an image that's going to run redis server whenever it starts up.

Create a file named: Dockerfile without extension.

The # symbol is named pound symbol.

After creating and wrting the instructions in Dockerfile, run: 
```shell 
docker build . .
```
Then copy the returned id from that command and then run:
```shell
docker run <returned id of the previous command>
```
Now in case of building a redis-server container, you should see: "ready to accept connections".

So we went through the process of creating the Dockerfile, using it to build an image and we started that image up.

## ??-030 Dockerfile Teardown:
In our dockerfile, every line started off with a single word that we refer to as an instruction. This instruction is telling the docker server to do some
very specific preparation step on the image that we are trying to create.

3 most used docker instructions:

---

The instruction `from`, is used to specify the docker image that we want to use as a base.

The `run` instruction is used to execute some commands while we are preparing our custom image.

The `CMD` command which stands for command, specifies what should be executed when our image is used to start up a brand new container.

---
Every line of configuration that you are ever going to add to a docker file, is always gonna start off with an instruction.

After each instruction, we then provide and argument to the instruction that kinda customize how that instruction was executed.

So for From instruction, the argument is `alpine`, or for RUN instruction, it's argument is: `apk add --update redis`.

## ??-031 What's a Base Image:
Writing a dockerfile is a little bit like being given a computer, a brand new computer with no operating system on it, and being told to
install google chrome on there. So:

**Note:** writing a dockerfile == being given a computer with no OS and being told to install chrome

The first thing we need to do on an empty computer is to install an OS.

When we specify that base image of alpine(in this case), that was a lot like installing an initial OS.
The purpose of specifying a base image is to kinda give us an initial starting point or an initial set of programs that we can use to 
further customize our image. 
So in `FROM alpine`, we said that we wanted to use the alpine docker image as kinda an initial OS or a starting point for the image that we are
creating.

Why did we use alpine as a base image?
Answer: Why do you use windows, MACOS or ubuntu or ...?
The answer is that you use any particular operating system because it kinda suits your needs. In other words, because the OS comes with a 
preinstalled set of programs that are useful to you.

In this case, we're trying to install and run redis and the alpine base image has a set of programs inside of it that are useful for installing and 
running redis.
The program that was most useful for us for installing and running redis, was found on the second instruction of dockerfile which is
the apk program which is a package manager. So we made use of alpine OS because it had this package manager automatically included inside of it.
So it made it a convenient thing to start with, inside of our dockerfile. 

## ??-032 The Build Process in Detail:
In docker build . , the dot is specifying the build context. The build context is the set of files and folders that belong to our project.
It's a set of files and folders that we want to kinda encapsulate or wrap in this container.

For every line of configuration we put into the dockerfile, we got an additional step added to the build flow when you run docker build <context>. 

In the first step that we said: FROM alpine, the first thing that occurred, was the docker-server looked at our local build cache and it checked to
see if it has ever downloaded an image called alpine before. If not, docker server would reach out to docker-hub which is a repo of existing 
docker images that are free for public use.

The step that has: RUN apk add --update redis , would first cause a log to the console that says:
`Running in <an id>`
and a little bit later in console, it says:
`Removing intermidate container <the same id in the previous line that I wrote>`
This id is an id of a container.
We see these two logs also for the logs of CMD instruction. But there isn't any intermediate container log for the step 1.
So it appears that for every instruction that we added to the dockerfile, besides the first one, it appears that some like container of sorts was
created.

From alpine will get alpine image. Remember that an image has a filesystem snapshot and some start up command and at this point, we don't know what
the start up command for alpine is, but we don't care that much.

This is what happens behind the scenes of: RUN apk add --update redis:
When docker-server saw this line of configuration, it looked back at the last step that just occurred(which in this case is FROM alpine).
It looked at the image that came out of that previous step which is alpine image. Then on the RUN ... line, it took that image and created a new
container out of it. So in memory, we temporarily got a brand new container created at the very start of step number two and that's what
`Running in <container id>` means. It means it created a temporary container out of the image that was sourced during the previous step.
So we can kinda imagine that we took the entire filesystem snapshot from the alpine image and we stuffed it into this very temporary container that
was created just FOR the second instruction.
After creating this temporary container, the apk add --update redis command was executed inside that container as it's primary running process.
So we took the command of RUN command and we executed it as a process inside of the container.

Note: apk is a package manager that is built into the alpine image.
it would download and install redis and a couple of deps for redis and you can imagine during that installation process, we maybe got like a new folder on the 
container's hard drive and that folder can be called for example redis.

After that package was installed, we then stopped that contaienr, we took a filesystem snapshot of that container and then we stopped it entirely.
So when we have: `Removing intermidate container <the same id in the previous line that I wrote>` , we're saying that temporary container that was 
just created, we stop it and we then take it's current filesystem snapshot and we save it as a temporary image with the id that it logs.
So the output of everything inside of step number 2 , is a new image that contains just the changes that we made during this step.
So the id that it logs after step 2 is finished, is the id to a very temporary image. So NOW, instead of a default alpine image, we have a more customized image
with that id that was logged which in this case, inside that image, it now has a installed copy of redis and we also throw away that intermediate container that we needed
in order to install redis. But we throw it away, because we just need a snapshot of it's filesystem AFTER running the RUN instruction which changes that image a bit.

In total in step 2, we created a temporary container out of the alpine image, we executed that apk add ... command inside tht container and then we took that 
container's filesystem and saved it as a temporary image and we remove that temporary container. So now we have this temporary filesystem snapshot that has redis included.

With CMD, we're gonna look at the image that was generated during the previous step. We create a new very temporary container out of it.
When we make that temp container, we take the image's filesystem snapshot and stuff it into that new temp container and then with CMD,
it is setting the primary command or the primary process of the container.
So the container doesn't actually execute the command we put for CMD. It just tells the container: Hey, if you were to ever run for real, you should be running this 
command(which in this case is `redis-server`) as your primary command. Then it shuts down that container and takes a snapshot of it's filesystem and it's primary command.
At the end of this step, we remove the intermidate container. We take a snapshot of it's filesystem and it's primary command(that should be executed when the container gets really 
started by us in the future) and then we save it as an output which is an image with the logged id as the id of image.

At the end, we get a final image that was created out of this entire series of steps which are defined by the dockerfile.

So along every step in the flow(not every, because in FROM step, we didn't have this), we take the image that was generated during the previous step, we create a new container out of it,
we execute a command in the container or make a change to it's filesystem. Then we take a snapshot of it's filesystem(or in some cases, we would also take a snapshot of it's primary command which 
would create a new process which would have a long life or not) and save it as an ouput for the next instruction along the chain and when there is no more instruction, the image that was 
generated during the last step is output from the entire process as the final image that we really care about. 

## ??-033 A Brief Recap:
The FROM alpine instruction told docker-server or docker-damon to go and download the alpine image(if we don't have it in the build cache) and we use the alpine iamge as a base because 
it comes preinstalled with some handy programs that ar useful for us creating our image.
The next thing that occurred was the RUN instruction. The first thing that occurred in that instruction, was it looked for the image from the previous step and so it went back to the previous step
and got the alpine image that had just been downloaded. That image was then used to create a brand new, very temporary container and then the command that we specified for RUN instruction, 
was executed inside of that temp container. The apk program was executed, it downloaded and installed redis inside the filesystem of that container.
So the result is a container with a modified filesystem and now has redis added onto it. 
We then took a snapshot of that container's new filesystem and the result was a filesystem snapshot, We then shut down the temp container that had just been created and we 
got our image ready for the next instruction and the image that we're using, is that filesystem snapshot, 
So you can imagine, that we're now passing onto the next step, the alpine image but with redis added on top.
In CMD ... instruction, we again get the image from the previous step, we again create a temp container out of it, but this time around, rather than executing a command inside
there, which is the purpose of the RUN instruction, we INSTEAD use the CMD instruction which is used to specify what the IMAGE(NOT container) should
do when it is started up as a container. So we told the temp container that we just created, that whenever it gets started in the future,
it should try to execute the command that we specified for the RUN instruction(redis-server in this case) and the result was a container that has a 
modified primary command.
We then shut down that temp container and we took an image out of it and got it ready for the next instruction.

But there are no more instructions, so the output of overall steps is whatever image that was generated during the last instruction inside of our dockerfile.

## ??-034 Rebuilds with Cache:
This tip gives docker more performance whenever creating a new image.

Note: From each instruction, we get a new image. So at every instruction along the way in dockerfile, we have a filesystem snapshot and a startup command for that image.

If you have a cache for downloading and installing a dep in dockerfile, you won't see any: 'running in ...' or any 'fetch ...' , instead we see a single
message that says: 'Using cache'. It means docker has realized that from the previous step, nothing has changed from the last time that we ran
docker build. In other words, it knows that it's gonna get the same image from the previous step, because it's the exact same instruction that it was before.
If we have a cache for an image, rather than going through the process of creating another container out that image and running
a command too inside that container again, it uses the image that it was generated during the previous step.

So anytime that we make a change to our dockerfile, we're gonna have to ONLY rerun the series of steps FROM THE CHANGED LINE <<ON DOWN>>.
So if you just change the ORDER of instructions in a dockerfile(not the actual instructions), it will use the cache version UNTIL arriving 
to the changed order line and after that, it will rerun the steps completely.
Because the last time, it ran those changed lines, it was done instruction 1 after instruction 2, but now their re-ordered, we have now 
instruction 2 and then instruction 1. So the cache cannot be used from those re-ordered lines down to the rest of the dockerfile.

So if you ever expect to have to change your dockerfile, you always want to put those changes as far as down to the end of the dockerfile as possible.

So by changing only the order of operations inside of our dockerfile, can dramatically change how long it takes to rebuild our image.

## ??-035 Tagging an Image:
At this point, we can create a new custom image out of our dockerfile and we can do so by running docker build . .
The output from all that is that last line that says: successfully built ... . It shows the id that was just created. So now we can create a 
container out of this built image, by copying the image id and running: docker run <image id> . Now when we run this command, an instance of our image
will be created and we'll get a new container.

Having to execute docker run <image id> is pain. We don't want to memorize an id. We can do the same by running:
```shell
docker build -t <tag> .
```
By using -t <tag of image> , it's gonna tag the image that is created and make it easier to refer to it by using a name.
The convention is: <my docker id>/<the name of the project you're working on>:<version>
Version is gonna be 'latest' whenever you're building the newest version of an image.

The build context is the source of all the files and folders that are going to be used to build your container out. So it specifies the 
directory of files/folders to use for the build.

**Note:** When you want to run the container out your image, you don't need to supply the version(the thing after colon in the tag of image).
So you can say: docker run <docker id>/<name of project>. If you don't put the version at the end, then the latest version of the image will be 
used by default.
Technically the version number on the very end, it is technically the tag.
The process of what we just did is tagging the image, but technically JUST the version on the end is the tag and everything else before it in that name 
which we supply to -t, is like the repository or the project name.

## ??-036 Quick Note for Windows Users:

## ??-037 Manual Image Generation with Docker Commit:
