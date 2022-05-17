# 04-04 - Orchestrating Collections of Services with Kubernetes
TODO: build the image and run the container for client app
## 62-001 Installing Kubernetes:
Kubernetes is a tool for running a bunch of different containers together. It will automatically manage the creation and management of all 
these different containers and it's also gonna make it easy for us to communicate between these containers as well.

We give kubernetes some configuration to describe how we want our containers to run and interact with each other.

There are tww possibilities for setting up kubernetes:
Either you're running docker for windows/mac, OR you're running docker-toolbox or linux.
Now for option 1, go to docker system tray icon, go to preferences>kubernetes>enable kubernetes .

For option 2, we need to install mini kube. Mini kube is a tool that will start up a kubernetes cluster on your local machine.

Docker toolbox is not stable.

## 63-002 IMPORTANT Note for Minikube and MicroK8s Users:
To make sure kubernetes is running successfully, open a new terminal window(if you already have one open) and run:
```shell
kubectl version
```

## 64-003 A Kubernetes Tour:
Open a NEW terminal window(it's important), even if you have one already open and run:
```shell
kubectl version
```
K8s to some degree is about stability, because we're using it to manage a tremendous aspect of our app.

Let's say we've got our posts service source code, we took 1) all those source files and 2) a Dockerfile and we fed them more or less into 
docker and out of that an image for our posts service came out. So we get an image for posts service.
We can now create as many instances of this image or containers as we call them when running them, as we wish. The image is a blueprint for how to create a 
post container. 
At some point in time, we're gonna want to create an instance of this posts image or a container.

The prerequisite is to have an image. Whenever we want to use k8s, we have to have a set of images that are already ready to go. So of course,
we're gonna want to create a container out of the image at some point in time and we're gonna want to deploy it into our k8s cluster.

A node in k8s cluster is a virtual machine. It is essentially a computer that's going to run some number of containers for us.
We're currently running k8s on our personal computer. If you're running k8s on your own computer, it's extremely likely that you're only
running ONE node by default. So you have one virtual machine. It's only once we start deploying to some cloud provider that we're gonna have
MULTIPLE nodes.

To create a container out of posts image, we're gonna first create a configuration file. In this config file, we're gonna list out some explicit 
directions for k8s. For example in that file we say: please create two copies of the posts image for us, make containers and host them somewhere on
those nodes. Inside that config file, we're also gonna say: Please allow these copies of posts to be accessible. In other words, set up some 
networking on these things. Allow other containers to get created among these nodes to access these posts services that we're about to create.
So we're gonna feed that config file into k8s using command line tool named kubectl . That's how we interact with our k8s cluster.
The k8s cluster reads this config file and say: OK, I got to create two containers out of posts. So k8s is gonna look at among the world and 
try to find a copy of this posts image. It's going to first take a look at the docker damon or the docker server running on your personal computer.
On your computer, we have built up that post image already. So k8s is going to look at your local machine and see if there is a copy of that
image already available.
If it is not available, it will default to looking out at docker hub and try to find a copy of that image there instead.
Either way, k8s is gonna try to find that image. Once it finds that image, that's going to create two(in our case) containers, because we asked
for two and randomly distribute(more or less randomly, again there's some science to this), distribute those two containers among those nodes.

**Learn**: Each container that is created is going to be hosted or essentially created inside of sth called a pod.
**Note**: For the purposes of this course, we're gonna use the terms pods and containers INTERCHANGEABLY.
IN reality, pods and containers are not the same thing.
**Learn**: A pod wraps up a container and a pod can have multiple containers inside of it.

We asked for two copies of the posts service, so k8s is gonna create two pods, each of them is going to have a copy of the posts container running inside
of it.

In order to manage these pods, k8s is also gonna create sth called a deployment. This deployment is gonna read in that config file and it's gonna say: 
"oh, they wanted two copies of the posts image". So this deployment is going to be in charge of managing both those pods. If anything ever goes 
wrong with these pods? So if it crashes, or if it just stops running?
This deployment is going to make sure that that pod automatically gets recreated for us. So the deployment is in charge of making sure that 
we've always got the right number of pods and that these pods are running successfully.

Second step:
In that config file we had said: "Please allow us to access these services(in this case, containers running posts image) from other pods or containers, inside of 
our k8s cluster".
To do so, k8s is gonna create sth called a service.
The term `service` in the world of k8s is not like the way that we've been using the word service throughout this course. It is not necessarily a running program.
It's not necessarily a running server or a running application or anything like that. Instead, a service in k8s is sth that gives us access to running pods
or running containers inside of our cluster.
**Learn**: So the service is what is in charge of kinda abstracting away all of those difficulties we spoke about a little bit ago and handling networking among different
microservices.
Remember the event-bus we had which was challenging for us to have the event bus always know HOW to reach out to all those different copies of comments service(containers) that
exist and how to KNOW what port to reach out to, as well. Now we introduced a thing that says: "Just send me a req and I will forward it ont to the 
appropriate container." So that's what this service in k8s is. It abstracts away all the difficulties of trying to figure out what IP or waht port, some given program
is running on.

So if in some point of time in future, we create another pod, let's say running our event-bus , the event bus to reach out to posts containers, rather than
reach out to that containers DIRECTLY, it's going to instead just reach out to the k8s service which will be available at an easy URL.
So this k8s service is gonna make it easy for us to get from our event-bus and reach on out to our running pods. 

## 65-004 Important Kubernetes Terminology:
- k8s cluster: It's a collection of nodes + a master to manage all those different nodes. Effectively, it's the entire set of infrastructure that's going to run our code.
- node: a node is a virtual machine that's going to run all the different containers that we throw at our k8s cluster. A single cluster might have many, many nodes assigned
  to it. It might only have one node. So right now, on our local computer, we're running a k8s cluster and by default, it has just one node,
- pod: in the context of this course, we're gonna kinda use the term pod and container interchangeably. Technically a pod is sth that kinda wraps up a container and a 
  pod can wrap up multiple different containers together. For us in this course, there's really a one to one mapping between pods and container.
- deployment: deployments are gonna monitor a set of identical pods or pods that are meant to run the same container inside them. If anything ever happens to those 
  pods, for example if the pod crashes or just stops running for any reason, the deployment is gonna see that and automatically restart that pod for us.
- service: A K8S service(not microservice service), is sth that provides easy to remember URL, so that other running pods or containers inside of our cluster can easily
  access another pod. There are several types of services 

Note: We're in a course about micro-services, we're creating services that are programs, but we're also creating services in k8s that are sth totally different.
Now when we say service, what do we mean? Application service? k8s service? 
When we say service, it's always in the context of either k8s or some actual program that we're trying to run.

## 66-005 Notes on Config Files:
Let's look at config files we're gonna write to create actual pods, services and deployments. 

K8S config files:
- tell k8s about the different deployments, pods and services(referred to as 'Objects') that we want to create inside of our cluster. By the way, collectively,
  we refer to deployments, pods and services and other similar things that we create in k8s, as Objects. So in other words, we're creating config files, to create and 
  configure Objects.
- Whenever you write a config file, we're always gonna store these files with our projects. In other words, we're gonna commit these to Git and make sure we store them in
  some kind of source control. The reason for that is that these config files that we write, are really documentation of what our k8s cluster is doing.
  The config files can tell another engineer who starts to look at your code, about the different deployments, services and pods that you have created and it's really the 
  best documentation you're gonna have, to tell other enineer what your cluster is doing. 
- with the last point in mind, it's possible to create these object things(an object is deployment, pod, service) without using fake files. So we can create Objects
  WITHOUT config files, BUT DO NOT DO THIS. You should not create objects directly at the command line by writing in commands to create an Object. We're always going to
  write out these config files because they're gonna be a source of documentation to tell you and other people in the future, exactly what is going on inside your cluster.
  Because once you start to read the k8s docs, they're gonna tell you to run commands at your terminal to DIRECTLY create Objects without using any config file. ONLY follow
  k8s docs for testing purposes and learning purposes. DO NOT create resources or Objects directly at the terminal in a production environment. There are a 
  couple of exclusions to that rule. So use config files instead of directly using command lines. So use config files at all times!

## 67-006 Creating a Pod:

