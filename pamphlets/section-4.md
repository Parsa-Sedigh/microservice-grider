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
  best documentation you're gonna have, to tell other engineer what your cluster is doing. 
- with the last point in mind, it's possible to create these object things(an object is deployment, pod, service) without using fake files. So we can create Objects
  WITHOUT config files, BUT DO NOT DO THIS. You should not create objects directly at the command line by writing in commands to create an Object. We're always going to
  write out these config files because they're gonna be a source of documentation to tell you and other people in the future, exactly what is going on inside your cluster.
  Because once you start to read the k8s docs, they're gonna tell you to run commands at your terminal to DIRECTLY create Objects without using any config file. ONLY follow
  k8s docs for testing purposes and learning purposes. DO NOT create resources or Objects directly at the terminal in a production environment. There are a 
  couple of exclusions to that rule. So use config files instead of directly using command lines. So use config files at all times!

## 67-006 Creating a Pod:
In the first k8s config file, we're gonna create a pod directly, so just a pod by itself, out of our posts service.
For doing this, in posts directory, rebuild that posts service docker image and that's important that you REBUILD that image because we're
gonna apply an additional label to it which is gonna help us identify it inside the configuration that we're about to write. SO run:
```shell
docker build -t <your docker id>/posts:<version number like 0.0.1> .
```
We used . to say I want to build this image out of this current directory of posts that we're currently in. Now we can refer to that image by using
it's tag along with the version number.
Now let's write the config file to create a pod using that image. 

First create a new directory called infra(infrastructure), which is gonna contain all the configuration, all the code related to the deployment or management
of all of our different services.
```yaml
kind: Pod # we want to create a pod
metadata:
  name: posts # we want it's name to be posts
spec:
  containers: # inside that pod, we want there to be exactly one container and it's name should be posts and we're gonna build that container using the specified image name
    - name: posts
      image: parsa7899/posts:001
```
Now we want to use kubectl command line tool to make use of this config file. 
We're gonna tell k8s to use this config file to create a new object. Now cd to k8s folder and run command below to tell k8s that we want to use this 
config file that we're going to write it's name:
```shell
kubectl apply -f <name of the config file>.yaml
```
You will only ever see errors coming out of this command if there is a typo in the config file, unless you're not running the k8s cluster at all, so k8s is 
not running on your machine.

To inspect the state of to figure out exactly what is ging on inside of our cluster, there are a variety of different commands we're gonna run with kubectl.

In this case, we want to take a look at all the different pods that are running inside of our cluster, currently there should be one in our case:
```shell
kubectl get pods
```
The line that says: 
READY
1/1
x/y means there are y copies of that pod that are trying to be executed and x copies are successfully running.
68-
That's how we create an object in k8s.

We applied the config file to k8s and k8s uses this config file to decide what to do inside the cluster.

## 68-007 ErrImagePull, ErrImageNeverPull and ImagePullBackoff Errors
## 69-008 Understanding a Pod Spec:
- apiVersion: v1 : It turns out k8s is really extensible. When you install k8s on your local machine, it comes with a pre-set list of different
  objects we can create. So we can create things like pods, deployments, services and ... . We can also tell k8s about CUSTOM objects that we want to 
  create as well. The goal of apiVersion is to tell k8s about the pool of objects that we want to draw from. In this case, we're telling k8s: "look at the
  default `v1` k8s list of objects that we can create."
  So k8s is extensible - we can add in our own custom objects. `apiVersion: v1` specifies the set of objects we want k8s to look at.
- kind: Pod : With this, we specify the exact type of object that we want to create from that pool. In this case we want to create an object of type Pod and 
  remember a pod is gonna wrap up a container and for our purposes, a pod is more or less like a container.
- metadata: metadata is gonna include some different options, so we wanna apply to the object we're about to create. The most common option that 
  we're gonna apply is the `name` property. So by saying name: posts in the metadata , we're saying that when this pod is created, we want the pod to be given a 
  name of posts. With this given name to that pod, if we now run: 
  ```shell
  k get pods
  ```
  we see a pod named `posts` and this posts name is the metadata's name property.
- spec: is gonna have a list of configuration options for the pod that we're about to create. This is a very precise defintion that controls exactly what
  should be going on inside this pod and how the pod should behave.
  - The only required property we have to put in the spec, is containers. Containers is gonna be an array. That's what the dash at the beginning of it meant.
  - A dash means that we want ti add in a array entity. So we could technically have many containers inside that one pod, but in our case, we have just one. So one 
    array entry which in this case consists of a name and image. So the one array entry in containers in our case is:
      - name: ...
        image: ...
      - For that one array entry, we're gonna provide some configuration for that one container that's going to be created inside the pod. The container that's gonna be 
        created will be given a name(in this case posts) as well.
      - Note: The name properties don't really have a lot of big importance. You can provide a name of whatever makes sense to you. In this case, we gave the pod(with the
        metadata name's property) and the container inside of it(by using the name property in containers property of spec) an identical name. Is that bad practice?
        Not really, because our pod is only gonna be running one container(in our case). So it's kinda ok to have the pod and container have the exact same name.
        The only real purpose of a name for us in the context of this course is to help us with debugging and understand exactly what pod we're looking at or 
        what container we're working with.
      - We also provide the `image` property and this is gonna be the exact tag applied to the image. 
        - Note: If we do not provide a version at the very end, then docker will default to putting on `latest` in al scenarios(...:latest). If you ever apply or tried to 
          create a pod that has :latest on the end or doesn't have anything which implies :latest , then k8s default behavior is to try to reach out to docker hub and find this
          image listed on docker hub. If you haven't pushed that image to docker-hub, you would get an error.
          By putting on an exact version, k8s is gonna say: I'm gonna assume that if this image is located on the computer that I'm running on, I'm just gonna
          use that image directly. In other words, when it's :latest or an IMPLICIT :latest , which means that we don't have anything at all, k8s assumes that it 
          needs to go out to docker-hub and try to find the latest image as opposed to using the image that is already on your computer. So by putting on that 
          version, we're just saying: hey, if you find that version on the local computer, just use it, don't try to reach out to docker hub.
        - Usually when we write a pod spec or a deployment spec, we're usually just gonna specify the image in the <docker id>/<name> and not worry too much 
          about the version. So don't worry too much about putting the version here. We put the version to avoid errors coming up on the FIRST pod that we're creating.
  
## 70-009 Common Kubectl Commands:
A lot of docker commands translate directly into the k8s world.
Remember: Docker in general is just about running individual containers. K8s is about running A BUNCH OF containers TOGETHER.
Once we start making use of k8s, we're not raelly gonna use the docker command line tool much anymore. Instead, we're gonna use the command line tool that
we use to interact with k8s which is kubectl.

    docker world commands                                     k8s world
        docker ps                             ----->             kubectl get pods 1
        docker exec -it [container id] [cmd]  ----->             kubectl exec -it [pod_name] [cmd]  2
        docker logs [container id]            ----->             kubectl logs [pod_name] 3
                                                                 kubectl delete pod [pod_name] 4
                                                                 kubectl apply -f [config file name] 5
                                                                 kubectl describe pod [pod_name]
1: Print out information about all of the running pods
2: Execute the given command in a running pod
3: Print out logs from the given pool.
4: Delete the given pod.
5: Tells k8s to process the config
6: Print out some information about the running pod. Most importantly, it's gonna give us some debug information if we feel as though sth is going wrong with
the pod.

How we can create a pod?
We do so by processing a config file which is done by the fourth command. So to process a config file and create all the pods designated inside there, we use
kubectl apply -f and then the name of the config file with the yaml extension.

We could start up a shell inside of a container that has been ran inside of a pod with:
```shell
kubectl exec -it [name of the pod, like posts] sh
```

Remember a pod can technically run more than one container inside of it at a time, if you ever are running multiple containers inside of a pod and run this
kubectl exec ... command, kubectl will prompt you and ask you which container you want to run that command inside of. But again, in this course, we're ever gonna
run one container inside of a pod at a time, so we don't really have to worry about that scenario.

Note: For exiting from the prompt after running `kubectl exec -it ...` , you can write: exit to quit.

When you run the command to delete a pod, although it says: "pod <pod name> deleted", it might take a couple of seconds for the deletion to actually be processed,
all the way up to 10, so if it hangs for about 10 seconds or so, that's totally OK.
After deletion of pod, you can run: `kubectl get pods` and we can recreate that deleted pod by running: `kubectl apply -f posts.yaml` , then run 
another get pods command to confirm the pod was created from the config file.

After running `kubectl describe pod [pod name]`, look at the bottom and at `Events` section which is the events log. So if sth is going wrong with the pod, 
you will quickly want to describe the pod and look at the Events list.

## 71-010 A Time-Saving Alias:
Let's set up an alias on ur computer that anytime we write `k`, it will be automatically translated into kubectl. 
The exact process of how you do this, is gonna vary based upon your computer setup. For example we can use a shell called zshell. This means that the
program that is running inside of my terminal and interpreting the commands that I run, that program is called zshell. You might be using sth called bash instead.
On zshell for making an alias do:
1- open ~/.zshrc (or on bash, open ~/.bashrc)
2- add: alias k="kubectl"

## 72-011 Introducing Deployments:
Turns out we usually do not create pods in the style of config file we did in the last vid.

In the world of k8s, we have pods and those are intended to run some container which is running an image. Rather than creating these pods DIRECTLY however,
we're usually gonna create sth called a deployment.
**Learn**: A deployment is a k8s object that is intended to manage a set of pods. Each of these pods are gonna be identical in nature, that they will all be 
running the SAME CONFIGURATION, SAME CONTAINER inside them.
The deployment has two big jobs assigned to it:
1) The deployment first off is gonna make sure that if any pod mysteriously just disappears for any reason, in other words, if the pod crashes for any reason,
   then the deployment will automatically create that pod for us again. The deployment is gonna make sure that if we tell it: make sure there are 3 pods available running
   the image, deployment is gonna say: I got it. I'm gonna make sure there are always exactly three pods running that image. That's the primary job of the deployment.
2) The deployment is useful for this reason too: At some point in time, we're gonna decide to update the code inside of our different containers. In other words,
   we might start off with all of our pods running a v1 of some application we put together. But then at some time we might tell the deployment that we want
   to roll out a new version, like v2. The deployment is gonna take care of this update for us automatically. Behind the scenes, the deploument is gonna create
   some number of new pods running this new version of our application. After these pods are up and running succesfully, the deployment is then gonna start to 
   manage that set of pods. So it's gonna say: I'm now gonna manage these 3 pods and the deployment will start to sunset of essentially delete the old pods for us
   automatically. So all of those old pods are gonna fade away once the new ones are up and running.
   So we can think of the deployment as just being a manager, it manages a set of pods. 

Understanding how to work with a deployment comes down to writing a config file and the commands we write in config file about deployments, at most the only 
thing that we're ever gonna do with them is to list out the deployments we have and then try to inspect their status. 
Deployments don't actually run any of our code or anything like that, so we're not really gonna ever have to try to run some command in the context of the 
deployment or really try to fetch some logs from it or sth like that. Instead, we really just want to list them out, take a look at their status.

So we want to create a deployment by writing a config file and tell it to run our posts application inside of a set of pods.

## 73-012 Creating a Deployment:

