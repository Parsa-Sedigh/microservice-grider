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
If your pods are showing ErrImagePull, ErrImageNeverPull, or ImagePullBackOff errors after running kubectl apply, the simplest solution is to provide an imagePullPolicy to the pod.

First, run kubectl delete -f infra/k8s/

Then, update your pod manifest:

spec:
containers:
- name: posts
  image: cygnet/posts:0.0.1
  imagePullPolicy: Never
Then, run kubectl apply -f infra/k8s/

This will ensure that Kubernetes will use the image built locally from your image cache instead of attempting to pull from a registry.

Minikube Users:

If you are using a vm driver, you will need to tell Kubernetes to use the Docker daemon running inside of the single node cluster instead of the host.

Run the following command:

eval $(minikube docker-env)

Note - This command will need to be repeated anytime you close and restart the terminal session.

Afterward, you can build your image:

docker build -t USERNAME/REPO .

Update, your pod manifest as shown above and then run:

kubectl apply -f infra/k8s/

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
Let's delete the posts.yml, if you want to, you can backup the file or just change the extension. You want to make sure you don't have that file anymore.
I wanted to keep a backup of it, so I renamed it to posts.yml.old . 
So we're gonna delete that file(or change it's extension), because we don't want to create an individual pod anymore. 
Then create posts.depl.yml .
The `depl` indicates that this file is gonna create a deployment. Inside this file, we're gonna say that we want to run some number of pods that are going to
run our posts image.

Note: There are certain buckets of different types of objects we have access to. In this case, the deployment is inside this bucket of different objects called apps/v1.
A pod is just inside of v1.

The `spec` section in config file specifies how this object should behave.

`replicas` is the number of pods we want to create which would run some particular image. 

These two parts are really working together:
selector:
    matchLabels:
       app: posts

metadata:
    labels:
      app: posts

A deployment kinda has a hard time figuring out which pods it should manage inside of our cluster. In other words, if a set of pods get created,
we have to somehow give the deployment some information and tell it: Hey, these pods over here, these are the ones you need to manage and that's what the 
goal of that
selector:
    matchLabels:
     app: posts
and
metadata:
    labels:
      app: posts
is all about.
The
selector:
    matchLabels:
      app: posts
is saying: take a look at all the different pods that have been created. Find all the pods with a label of `app: posts`. So we refer `app: posts` as a label.
It's a key-value pair that we're just applying some arbitray information to a pod. In `app: posts` , app doesn't really have any special meaning.
Wwe can sset up labels with any structure you can imagine, so we can have a label of: asdasds: qweqweqweqwe .

So when we have:
selector:
    matchLabels:
        app: posts
we're telling deployment that we wanted to find all the different pods with the label of `app: posts` and those are the pods that it should be in charge of.

Nww when we have: 
template:
    metadata:
        labels:
            app: posts
the template is where we specify the exact configuration of a pod that we want this deployment to create. So this:
metadata:
    labels:
        app: posts
is some configuration that is going to be applied to the pod that the deployment is going to make, The above configuration says that we want the pod 
to have a label of app: posts .
So again this part:
selector:
    matchLabels:
        app: posts
and this part:
metadata:
    labels:
        app: posts
are working together to tell the deployment about which pods it should try to manage.
Everything inside the `template` key can be though of as a pod spec or a pod config file, similar to the pod config file we had before.
The `spec` section is describing the specification or the configuration options where the pod that we're trying to make.

The `containers` key needs an array as value and in yaml we designate an array entry with a single dash.

Anytime we want to make a change to our cluster, we create those config files and we will run that cubectl apply command.
Now to apply that config file, run:
```shell
kubectl apply -f <name of config file like: posts.depl.yaml>
```
Remember to run the above command inside the k8s directory in our project where we have that yaml file.

Now the deployment has been created, but what does that really do for us?

## 74-013 Common Commands Around Deployments:
To inspect a deployment and manipulate it, there are some commands:
`kubectl get deployments` : this is gonna list out all the runing deployments we have on our cluster
In the output of above command, `READY` is: 
<number of pods that are up and running and ready to receive some traffic or do some work> / <the number of pods that we're trying to create>
We also get the number of up to date pods as UP-TO-DATE, because remember we use deployments to manage updating the versions of some code or application running inside
these pods.
The AVAIlABLE output is the number of pods that are available and again, ready to do some work for us.

We can also print out the pods we have running inside of our cluster as well. We should see one pod that has been created automatically by this deployment in our 
project, so we can run:
```shell
kubectl get pods
```
and you should see the pod that was created by our deployment and it's NAME should be sth like: posts-depl-7ff... .
Now if you try to delete that pod that was created by the deployment, the deployment is gonna see that that pod has mysteriously disappeared and so it's gonna
attempt to recreate the pod for us.
Let's try to do this.
First delete the pod that was created by the deployment by running:
```shell
kubectl delete pod <name of the pod>
```
Then if you run kubectl get pods again, we still got that deleted pod!
(Well, not quite. The last part of the name of the deleted pod is changed(the name pattern is: <name of pod>-<id>-<another part of id>)!
Although it still shows up in `kubectl get pods` command.
In addition to the name change, the age of the deleted pod is not much. So the instant we deleted the old pod, our deployment automatically created a new
one for us. 
So you see how using deployments to manage some workloads inside of our application are really fantastic. If anything starts to go wrong, k8s has our back, 
that's going to automatically try to repair our application.

`kubectl describe deployments`: print out details about specific deployments. We use this anytime we want to debug a deployment.
The `Events` section in the output is gonna tell you about what is going on inside of your deployment.

`kubectl apply -f [config file name]`: We can create a deployment out of a config file with this command.

`kubectl delete deployment`: delete a deployment.
As soon as you delete a deployment, all the associated pods are gonna be deleted as well. So after running the delete command for a deployment, if 
you run `kubectl get pods`, the related pods to that deleted deployment should be deleted as well and we shouldn't see those pods.
We can then recreate the deployment with `kubectl apply -f <name of config file>` .
Then show the deployments' info: `kubectl get deployments` and also show all the pods to see the created pods related to that newly created deployment:
`kubectl get pods` .

Do not create pods manually because if they crash for any reason, then we don't have anything to start them back up.

## 75-014 Updating Deployments:
We use deployments because we can easily update the version image used by each of our different pods. For example if we make a change to our posts project and
we rebuild the image, we might want to redeploy our application and have our deployment run this new version 2 instead.

In this vid, we're gonna see 2 different ways of actually applying this update. So we're gonna look at method #1 to update the image used by a 
deployment. This method is not used often in professional deployments.
The steps for this approach:
1) make a change to your project code
2) rebuild the image, specifying a new image version
3) in the deployment config file, update the version of the image
4) run the command `kubectl apply -f [depl file name]`

Change sth in the posts service. Then rebuild the posts image and when we rebuild the image, we're going to tag that image with some new version number.
For example run:
```shell
docker buld -t <your docker id>/posts:0.0.2 .
```

Now we're bog a new image with an appropriate image version tied to it.
Now find the k8s deployment config file and update the version of the image which we just changed to a newer one, inside there.
So change image: parsa7899/posts:0.0.1 tp image: parsa7899/posts:0.0.2

Now we need to tell k8s to use that new updated config file. By just making a change to config file, nothing changes inside of our cluster. K8s is not
aware that this file(the config file) even exists. Whenever we make a change to a config file, we have to eat it back into the cluster using that same
kubectl apply -f <depl file name> .
Whenever you apply a config file, k8s is gonna know whether this is a config file that has ALREADY been fed into it before, or if it's sth brand new.
In the case of updating a config file and then applying it again, we have a config file that we've been already fed in once. So k8s is gonna know that we;re
trying to make a change to an existing object as opposed to creating a new object.
That's gonna take a look atall the different config options, it's gonna see in this case that the only thing has changed is the version image and so it's gonna
make that one single change to our existing deployment as opposed to creating a new deployment or sth like that.
So run:
```shell
kubectl apply -f <name of the deployment config file>
```
Notice after running this, it would print out for example: deployment.apps/posts-depl configured. As opposed to '... deployment was created'.
So k8s knows that this is a config file related to a deployment that already exists. But rather than creating a new deployment, it configured or changed the
configuration of the one that already exists.
Now run:
```shell
kubectl get deployments
```
then:
```shell
kubectl get pods
```
to see the newly restarted pod and it's age should be not much because it was just created using the new version of the image.

We're not gonna use this apporach frequently, because in the config file, we're specifying a precise version hard coded. So anytime we want to make a 
change to our app or redeploy our app, we would have to always open up that config file and make a change to that version number.

So we don't want to specify a version there at all, it's good to tell k8s to always use the latest version of a given image. This is for approach #2.
So we're gonna see a way to tell k8s to use the latest version of an image and we're gonna figure out a way to tell k8s to automatically update ITSELF or
re-setup our deployment anytime that version changes as well or anytime we create a new image as well.

## 76-015 Preferred Method for Updating Deployments:
We saw method #1 for making changes to a deployment. 

We would be well served to not have to specify the version of an image inside of our deployment config file and the reason for that is that if we have to ever
open up the config file to change the version of some image, that's always a possibility of introducing some kind of error. Like make a typo or ... .
So it would be great if we could figure out some way to now have to specify the version whatsoever in the k8s config file.

1) Make sure that our deployment is using the 'latest' tag in the pod spec section
2) Make an update to your code
3) rebuild the image
4) push the image to docker hub
5) run `kubectl rollout restart deployment [depl_name]`

- step #1, where you have:
  spec:
  containers:
    - name: posts
      image: parsa7899/posts:0.0.1
      Change 0.0.1 to either latest, or just leave off that designation entirely which means docker assumes that we want to use 'latest'. I just leave it off.
      Now remember just saving that config file with some changes, does nothing whatsover. We have to take that changed config file and apply it to our cluster. So for example
      our deployment inside of our cluster understands that it should try to use the latest version image.
      So inside the k8s directory, run:
```shell
kubectl apply -f <name of the config file with yaml extension>
```
to UPDATE or CONFIGURING the already running deployment.
Now at this point in time, if you do a `kubectl get deployments` and you see that we're not running one pod or if you see an error or anything like that, it's ok.
- step #2: make a change to posts service code.
  To rebiuld the image: Go to the directory where the dockerfile exists and run: `docker build -t <dokcer id>/posts or whatever .` .
  **Important**: The dot in the above command is to designate that we want to use all the code inside of that directory where we're running that command.
- step #3: run:
  ```shell
    docker push <you docker id>/posts
  ```
- step #4
- step #5: We need to somehow tell our deployment to use that latest version. So run the specified command in the step #5 above.
  In this step, first run: `kubectl get deployments` to get the name of the deployment you want to update. Then run the specified command in step #5.
  After running the rollout command, run the kubectl get pods to see if that related pod was just created.
  Now we need to get logs out of that pod and make sure that we see the changed code(latest version of code). So run:
  ```shell
    kubectl logs <name of pod>
  ```

So we saw two methods to UPDATE THE IMAGE USED BY A DEPLOYMENT.
Method #1 is gonna require us to specify a version number inside of our config file, which is not good.
In method #2 , we always use the latest tag in the config file, make an update to our code, push the image to docker hub and then run the rollout command and after that,
our cluster is now running the latest version of all of our code(in other words, is using the latest docker image).

## 77-016 Networking With Services
Currently, we have a pod that inside of it, it's running a container and inside that container, is our posts application. Now how do we make a 
request to that application?
We're gonna discussing services.
A service is another kind of object in k8s. It's an object similar to how a pod is an object or a deployment is an object. We create services using config files just 
as we do pods and deployments. 

**Important:** We're gonna use services to set up some communication between all of our different pods or to get access to a pod from the outside of our cluster.
So anytime we need to have some communication or networking between our pods, which is the case with our event-bus trying to send events over to posts images(actually posts pods),
or anytime that we need to somehow access a pod from outside of the cluster(like sending a req to an application inside of a pod which is inside of a service), we're always 
gonna be making use of a service.
So anytime we think of networking or communication in any way, we're always thinking about services(k8s services).

Just about every single pod or every deployment that we ever create, we're gonna want to communicate with, in some fashion. So just about every pod or every deployment
we create, is always gonna have some kind of matching service along with it.

Behind the scenes, there are several different kinds of services.

### types of services:
We're only gonna use cluster IP and load balancer on daily basis.

- Cluster IP: Cluster IP services are used anytime that we want to set up communication between different pods inside of our cluster. This type of service,
  sets up an easy-to-remember URL to access a pod. Only exposes pods in the cluster.
- Node port: These are used any time we want to access a pod from outside of our cluster, but we usually only use a node port for development purporses.
  So this service makes a pod accessible from _outside the cluster_.
- Load balancer: These are the right way to access a pod from outside the cluster, so you can think of a node port and load balancer as being very similar, but
  the way that they actually work are very different.
  So node port and load balancer are both going to expose a pod to the outside world.
- External name: These services is an advanced thing. Redirects an in-cluster request to a CNAME url.

What is the most appropriate type of service to use if we want to set up some kind of communication between these different pods?
Well, in this case we're trying to set up communication between pods INSIDE of a cluster. So cluster IP type of service is best. Because it only exposes pods to 
other pods inside the cluster.

How about the other scenario? Scenario in which we want to access a pod from the outside world, like maybe a browser running on our computer or a browser running on some 
user's machine?
We could use either a node port or a load balancer. If we're in a development environment and we just want to do one little quick one off test, we would use a node port.
But in just about every other scenario, we're going to use a load balancer type service.

## 77-017 Creating a NodePort Service
Why are we making a node port first, if it's not what we're gonna use often?
Because a cluster IP is used to network different pods together. Right now we have only ONE pod. So if we start to create a cluster IP service, it's not really gonna do 
anything because we're not really exposing our pod to any other pods because we don't have any other pods.
Why are we not creating a load balancer?
A load balancer usually requires a decent amount of additional configuration.

So in this case that we only have one pod, the easiest service to create and the one that actually does sth for right now, or actually has some effect on our cluster, is 
a node port. So let's create it.

All of the 4 different services are types of objects. A deployment, a pod, a service, are all referred to as objects in k8s.
All objects we create, are made by first creating a config file and then applying that config file to our cluster.
So whenever we talk about creating a service or modifying a service, we're always talking about creating a config file.
Now create posts-srv.yaml .

The spec key is gonna customize how that service behaves.

What is the `selector` in k8s config file?
Remember the entire goal of a node port is to expose a set of pods to the outside world. That node port service needs to know about what pods it should actually expose and
that's the goal of that `selector` key.
When we have:
selector: 
    app: posts
we did sth similar to this in the posts-depl.yaml . The labels key in that posts-depl.yaml is applying a label or some kind of designation or an identifier to the pod that
is going to be created. 
**Note:** So by writing:
selector: 
    app: posts
we're telling the service to try to find all the different pods with a label of app: posts and expose traffic or expose those pods to the outside world.

This whole label system in k8s in genral, you can think of it as similar to class names that get applied to html elements. With class names in html, we can select a broad range or a 
set of elements easily.

The `ports` section in config file of that posts-srv.yaml is gonna list out all the different ports that we want to expose on a target pod.
Recall that in the posts service(posts application) we are using port 4000 . So inside of the k8s service we need to expose port 4000 for posts application.

`ports` is gonna be an array of entries and we set up an array in yaml by putting a dash in the beginning.

In the ports array, why we're listing port and targetPort? What is the difference between port and targetPort?
Look at img 78-018-1 .
Target port is the actual port that our application is listening for traffic on. That's where we ULTIMATELY want to send traffic to.
Now the node port service is gonna have a port of it's own and that port is simply referred to as 'port'.
So we're sending traffic to port 4000 on a node port service and that service internally is gonna redirect that traffic over to port 4000 on the container itself.
Usually, you're gonna have a port and a targetPort that are identical.

Another example: The comments application is listening on port 4001, so the port and nodePort keys of k8s config file for that service are gonna be 4001. 

## 78-018 Accessing NodePort Services:
Let's apply the posts-srv.yaml to cluster. To do so, go to the directory where that config file exists and run:
```shell
kubectl apply -f posts-srv.yaml
```

To list out all the services running inside our cluster: 
```shell
kubectl get services
```
There's always a default service with the name of kubernetes.

Currently, in the PORT(S) section of the above command you would see sth like 4000:30901/TCP where the second number is gonna be anything from 
30000 to 32000. This is gonna be a randomly assigned port and that port is how we're going to actually access your service.

In the last picture that I mentioned(78-018-1) we said that there is a targetPort that we use to access our container running inside the pod.
There is a `port` that gets assigned to the node port service itself and finally there's a third thing called a `nodePort`. The WORD nodePort is meant to 
designate a kind of port as opposed to the node port service. The nodePort port is randomly assigned port that we use to get access to that service from outside
of our cluster.
**Important:** So the nodePort is how we're gonna actually access the service FROM OUR BROWSER>
So it's almost always gonna start with 3xxxx. 

We can get that nodePort port by running `kubectl get services` and look at the PORT(S) section of the related service.
We can run:
```shell
kubectl describe service [service name]
```

Look at 78-018-2 image.
To actually access our posts pod, it comes down to whether you're using docker for mac or windows, or you're using minikube.
If you're using minikube to access the serve we just created, you're going to run:
```shell
minikube ip
```
which is gonna print out an ip address. You're then gonna take that IP and write: `<ip>:3xxxx/posts` which 3xxxx is the nodePort.
If you're on docker for mac or windows, just write: `localhost:3xxxx/posts`.

Note: This entire nodePort thing is all about development purposes. So this does not mean that your users are gonna actually access your application at 
port 3xxxx .

So now, for example, you can type `localhost:3xxxx` in your browser to access your application.  

We now know how to run a container inside of cluster and how to in some way access it by visiting some address inside of our browser.

Now ho to create a load balancer, which is a little bit better way of accessing stuff than node port and we also need to ultimately start to create some of 
the other pods for our other services(event-bus service, comments service and ...) and then get connections between all these different things using cluster IP services 
as well.

## 79-019 Setting Up Cluster IP Services:
**Note:** The goal of a cluster IP service is to expose a pod to other pods inside the cluster.

Let's create a new pod that is going to run event-bus , we're then going to set up some services to allow posts and event-bus to communicate with each other,
Look at 78-018-3 image.
In part of the application that is shown in the diagram, we have one node, we currently have a podd running our posts app and soon, we're gonna set up a second pod 
to run all the event-bus code. These two pods are gonna have to somehow communicate with each other. Now unfortunately, these pods CANNOT just reach out DIRECTLY to each other.
They technically can, but there are a variety of reasons that we don't do so. For example, we never really know a head of time what IP address a pod is going to be assigned and
so we can't just reach out to, say, localhost:xxxx . Because that pod will not be accessible at localhost. Instead, it gets assigned more or less a random 
ip address inside of our k8s cluster and that event-bus will not know A HEAD OF TIME, what that IP address(of posts pod) is. That's why we have to communicate between
these pods through those cluster IP services.

**Important:** So pods communicate with each other through each other's cluster IP service.

The posts pod is gonna want to reach out and send a message over to the event-bus, but we can't reach out DIRECTLY. Instead, we're gonna create a cluster IP service that's
gonna govern access to the event-bus pod.

## 80-020 Building a Deployment for the Event Bus:
So now we need to create another deployment that's gonna create that event-bus pod. Then create one cluster IP service for posts and one for event-bus.

To get event-bus and posts pods working together, we got to do:
1) build an image for event-bus
2) push the image to docker hub
3) create a deployment for event-bus(that deployment is gonna create one pod for us automatically)
4) create a cluster IP service for event-bus and posts(the cluster IP services are gonna teach those pods how to access each other inside the cluster)
5) write it all up!

To do this, go to event-bus directory and run:
```shell
docker build -t <docker id>/event-bus . # dot means we want to use current working directory for all the source code inside the image we're gonna build
```
Now push the image up to docker hub so that the deployment can easily pull down that image when it decides that it's time to create the pod. So run(better to be in
the directory where the dockerfile related to that built image, exists):
```shell
docker push <name of the built image>
```
Now for creating deployment for event-bus, we're gonna create a config file. So create event-bus-depl.yaml .
Then apply that configuration file to k8s using kubectl. So first cd to k8s directory and run:
```shell
kubectl apply -f event-bus-depl.yaml
```
Then it should say: deployment.apps/event-bus-depl created
So the deployment is created and that should automatically create the pod for us.
Let's just list out all of the different pods and make sure that a new pod related to event-bus was created:
```shell
kubectl get pods
```

Currently, we have no effective way to have any communication between that event-bus pod and the posts pod and so that is why we need to add in
cluster IP services for posts and event-bus pods. Again, they technically can communicate, but the IP address assigned to these pods is variable and we can't
predict it ahead of time. The cluster IP services are gonna give us a well known, easy to predict URL that we can type in any time we want to have our
event-bus communicate over to posts or vice-versa.

## 81-021 Adding ClusterIP Services:
We can either create two new files to house those two cluster IP services or alternatively, we can just write out the configuration for each cluster IP service directly into the
appropriate deployment file.

It's good to colocate our deployment config and the IP service that MAPS to it and the reason for that is that we usually end up with a ONE TO ONE MAPPING BETWEEN
CLUSTER IP SERVICES AND THE DEPLOYMENTS that they are allowing access to. So it makes sense to colocate all those stuff(deployment and cluster IP service).
Another approach is to create a separate file foe every single object inside their k8s cluster.

With approach #1, we write the config for cluster IP service inside the related k8s depl files.

Now because we wanna create multiple k8s objects inside a single yaml file, we add on `---`.

In the case of service object, the `selector` inside spec property is gonna tell the service what pods it is going to allow access to. 

In case of event-bus-srv , we wanna tell the service to direct any incoming traffic to the pod with a label of `app: event-bus` , so this is the label
that we're gonna look for in the event-bus service.

Next, we can optionally specify a type of `ClusterIP`. The reason I say optionally, is that if we do not specify a `type` for the SERVICE, k8s will default to 
creating a ClusterIP service for us. So we can remove that type: ClusterIP if we want to and we will still get a ClusterIP service.

Now about ports section of cluster ip for event-bus: We know that the event-bus APPLICATION is listening to incoming traffic on port 4005. So we specify a port
of 4005 in the port configuration of that cluster ip service.

Now apply the changed config file with k8s `apply` command.

Now run:
```shell
kubectl get services
```
You should have a event-bus-srv.

Now we need to repeat the same process for posts.
For posts, we already created a node port service for posts and it's already using the name `posts-srv`. So we just need to aware that once we create a second
service which is going to direct to that posts pod(so we would have a cluster ip service in addition to node port which already points to posts), we just need to use a 
slightly different name. Otherwise k8s is gonna think that we want to try to change the existing node port service into a cluster ip.

So in the posts-depl.yaml , create another object as a cluster ip service.

The posts application server is listening on port 4000. So we need to use that port for cluster ip service.
Then apply that changed config file with kubectl apply. Then run `kubectl get services`.

Now we need to write these stuff up and then our two pods(event-bus and posts) will be able to communicate with each other easily. 

## 82-022 How to Communicate Between Services:
When we want to access event-bus by writing: http://localhost:4005 , this only works correctly when we were running the event-bus on our local machine and
not inside of, say k8s. K8s right now technically is on our local machine but you know what I mean!
So we can no longer reach out to http://localhost:4005 to get at our event-bus. Instead, we need to figure out how we're gonna reach over and make a req to 
the cluster IP service of the other pod and hopefully that thing will forward the req on to the other pod.
So what is the url of other pod's related cluster IP service's URL address?
The url is gonna be the name of that cluster IP service. We can retrieve the exact name of that cluster IP service by running:
```shell
kubectl get services
```
For example, we can reach from posts pod to event-bus related cluster IP service with: http://event-bus-srv:4005 . Look at 78-018-4 .

**Note:** Anytime that we want to have some communication between pods, we're always gonna create a cluster IP service for the pod we're trying to reach out to.
Once we have created that cluster IP service, to make a req to it, we're gonna write a URL of http://<name of the cluster IP service> .

## 83-023 Updating Service Addresses:
After changing the urls, we need to rebuild the images and update our deployments, so that they are running that new changed code.

Currently if you try to send req from event-bus(which is inside a pod) to other services that aren't inside a pod, you would get an error. So you can comment
them for now.

We changed the urls of microservices, now we need to deploy the microservices that are using the right urls to send req to other services(microservice service).
In other words, we need to update deployments.
So update the deployment for event-bus and posts apps.
After updating both, run rollout command of kubectl to restart deployment for both deployments by running:
```shell
kubectl rollout restart deployment [name of deployment]
```

You can get the name of deployments by running: `kubectl get deployments`

Then list out all your pods and make sure that they both just got restart by looking at their AGE. If you run the command for getting all pods quickly enough, you may
see the old pod! Like it's in Terminating status and another one which is just like that is in Running status. So you may have more pods that the expected number of pods,
because the pods replaced with new ones and the old ones take some time to shut down and get replaced. So you may get n + 1 pods instead of n pods if you run the get pods
command quickly enough.

Now open up postman and manually create a post to see if the post microservice works.

## 84-024 Verifying Communication:
For verifying the communication between posts pod and event-bus pod, we're gonna send a req to posts application to create a post.
So send a req(try to access) to node port service that's governing access to the posts pod and we know that if we're trying to access
some pod that is running inside of our cluster, we can use a node port service and we typically only use those for development purposes
and to access node port, we have to use the randomly assigned port that usually starts with 3xxxx and that port is named nodePort port(look at 84-024-1)
and the actual URL that we're gonna use depends upon whether you're using docker for mac/windows or docker toolbox with minikube.
If you're using minikube, you will run minikube IP and that will give you the IP address that you're gonna make your req to, otherwise
if you're on docker for mac/windows , you will use localhost as the url. (Look at 78-018-2). (When you're using minikube, you will
not have localhost).
To get the port that we need to make the req to, we can rerun `kubectl get services` and look at the right hand side PORT of the service
you want to make a req to.
Put `Content-Type: application/json` at the header of req and make sure the body is selected as JSON in postman.
We can use a node port service and we typically only use those for development purposes
and to access node port, we have to use the randomly assigned port that usually starts with 3xxxx and that port is named nodePort port(look at 84-024-1)
and the actual URL that we're gonna use depends upon whether you're using docker for mac/windows or docker toolbox with minikube.
If you're using minikube, you will run minikube IP and that will give you the IP address that you're gonna make your req to, otherwise
if you're on docker for mac/windows , you will use localhost as the url. (Look at 78-018-2). (When you're using minikube, you will
not have localhost).
To get the port that we need to make the req to, we can rerun `kubectl get services` and look at the right side of colon of PORT column of the service
you want to make a req to.

Now look at the logs at posts pod and event-bus pod by first getting your pods and then look at the logs inside them:
```shell
kubectl get pods
kubectl logs [pod name from first command]
```

The flow for creating a post till now is like this:
When we created that post, our posts pod sent the message(post created event) to the event-bus through the event bus service(event-bus-srv). Then the event bus took
that event and emitted to all the different services that are running inside our cluster(currently we have only posts cluster ip service other than event-bus-srv).
That means we were able to send a message to the event bus and receive a message from event-bus as well.

Now we're gonna create deployments and cluster IP services for comments, moderation and ... .

## 85-025 Adding Query, Moderation and Comments:
Look at 85-025-1.
For each of our microservices, we're gonna do the steps mentioned in image.

Step 1 for comments micro service:
Replace urls that it sends reqs to. For example change http://localhost:4005 to http://event-bus-srv:4005 and other urls accordiong to our k8s services.

After doing step #1 for each of the services, now they are all attempting to reach out to the event-bus cluster ip service. 
Note: The port of cluster IP service for event-bus is 4005.

Now we need to go to the root of other services that don't have deployment set up and build images out of each of them and push them up to docker hub.
For example go to comments service and run:
```shell
docker build -t <docker id>/comments .
docker push <docker id>/comments
```
and repeat this for other microservices.

We have done 2 steps.

For step 3 for comments microservice:
Create a file called comments-depl.yaml and write the config. You can copy the event-bus-depl.yaml code which includes the config for depl and for related cluster IP service.
Don't forget about the right port to use.

Note: We have put the deployment and cluster IP services in one single file for our microservices.

After creating the depl and cluster IP config for each of our microservices, we need to apply those config files to cluster.
So run command below for all of the files that haven't been applied(first you need to cd to k8s directory). To apply multiple config files with
one single command(you must be in directory where the config files exist or pass the path to that folder in command instead of dot):
```shell
kubectl apply -f . 
```
Here dot means find all the different config files inside of the CURRENT WORKING DIRECTORY.

For verifying:
```shell
k get pods
```
All of them should have a STATUS of Running.

If you see a status other that Running, like `ErrImageBackOff` or `ErrImagePull, the first thing to do to debug it, is to copy the pod's name and run:
```shell
k describe pod [name of the pod]
```

To make sure services are created: `k get services`.

Now we need to update the event-bus to once again, send events to other services as well.

## 86-026 Testing Communication:
Update urls in event bus service code. So instead of localhost in url, we need to reach out to direct CLUSTER IP SERVICES. If we evenr forget the names of those
cluster ip services, run: `k get services`.

---
**Note:** After changing the code, rebuild it's image, then run `docker push <docker id>/name` then finally command below to update the related deployment:
`k rollout restart deployment [depl name]`. You can get the name of deployment to update by runningL `k get deployments`.
Then run `k get pods` and make sure the related pod was updated by looking at it's age. The old pod for it would have a STATUS of Terminated and new pod 
would have a status of Running with a little AGE. After some time the Terminated pod would get cleaned up.
So the commands would be:
```shell
docker build -t <docker id>/<some name> .
docker push <image name>
k rollout restart deployment [depl name]
k get pods #  for verifying
```
---

For testing, in postman send some req and see the logs inside the event-bus app pod.
To check logs: `k logs [name of the pod we want the logs from]`.

**Note:** Remember to use FROM node:18-alpine3.14 in your dockerfile if for example you're using node v18.

## 87-027 Load Balancer Services:
Now we need to integrate the react app into our cluster.

Our react application is actually the create react app development server. We're gonna house it inside of a pod, so we're gonna create a new docker image for(or ?)
our react application or the development server. Then we're gonna create a container inside of a pod and place it inside of our cluster.

What does the react dev server does?
Whenever a user opens up his browser and navigates to our app, we're gonna be attempting to connect to that react dev server. The only goal of that 
dev server is to take code that we write inside of our react app and then generate some HTML, JS and css files out of it. So during the initial navigation attempt
to our application running inside that cluster, the only thing that react dev server is doing, is returning some html, css and JS. What that react dev server is NOT
doing, is making any reqs at any point in time, over to posts, comments or ... . The actual req for data, are being issued from the user's browser. 
So we send over that html, css and js stuff, then the react app boots up inside the user's browser and then INSIDE of the browser, the react app is gonna try to 
reach out to the query service and get some posts and comments.
After serving up the initial html, css and js, the development server is no longer relevant whatsover. 

So no reqs are being issued from that dev server directly to our other pods. All those reqs are coming from the browser. Look at 87-027-1 .
This opens up a question: How are we gonna make sure that the react app can somehow make reqs to all of those different pods?
To make sure that the react app can reach out to all those pods and the containers running inside them, we have two options available to us.
One is pretty bad and we're not gonna do it which is we could create a node port service for posts, comments, query and moderation. Look at 87-027-2.
Remember that a node port service is gonna expose a pod to the outside world or to traffic outside the cluster. So inside of our react app, we could write out
some code that would make reqs to those different node port services and access the underlying pods to get all the information that is required.
The reason this is not a good approach, is that remember, when we create a node port service, we're essentially opening up a random port, we can't technically specify
an EXACT port and so if we ever changed a node port, there's the possibility that who knows, maybe the port that we get assigned would be different. So we would go 
back to the react app and change the port inside of our code and update it to reflect the new node port. There other other reasons as well.

Rather than trying to expose all our pods using node ports, we're gonna take a look at load balancer service. We use this approach in production env and in a dev
env.
Look at 87-027-3 . 

The goal of this load balancer service is to have one single point of entry into our entire cluster. Then we're gonna make sure that our react app is gonna make reqs to acccess
that load balancer service and then we're gonna place some logic(actual code or configuration) inside of that load balancer service, that's gonna take those incoming
reqs and then route them off to the appropriate cluster IP service that we have for each of our pods.

So the react app(req of the react app) is gonna reach out(send req) to load balancer, req gets forwared on to the appropriate pod(actually the appropriate cluster IP service of
that pod), response comes through the load balancer and back to the react app. 

## 88-028 Load Balancers and Ingress:
We said that we needed a load balancer service to somehow get traffic into all of our different pods.
When we talk about a load balancer service, we're really talking about two distinct things in the world of k8s(look at 88-028-1):
First off, there's sth in k8s, called a load balancer service. There is sth that is also very closely related that is referred to as ingress or 
an ingress controller. Technically an ingress and ingress controller are two different things, but for the purposes of this course, we're gonna kind of refer
to them with interchangeable terms.

- A load balancer service is gonna be sth that tells k8s or specifically our cluster, to reach out to it's provider and provision a load balancer.
  The goal of a load balancer service is to get traffic into a single pod.
- On the other hand, an ingress or ingress controller(again technically different things, but we're gonna use these terms interchangeably) is a pod that has a
  set of routing rules in it that are gonna distribute traffic to other services inside of our cluster.

Let's image we're running one singular pod called 'some pod' inside of our cluster. Our cluster at some point in the future(not now because it's on our 
local machine), is gonna be running on some cloud provider, like aws, google cloud. We're going to want to somehow get traffic or network reqs from the outside world
to some pod inside of our cluster. So how do we do that using a load balancer?
We would create a config file for a load balancer service. We would then feed that thing into our cluster using that same `kubectl apply` command.
Now a load balancer service is a very special little thing. Just about all the objects we've talked till now on k8s, has been all about stuff that gets created directly inside 
of our cluster. For example we have created services inside the cluster, pods inside the cluster, deployments, but a load balancer service is a little bit different.
A load balancer service is gonna tell our cluster to reach out to it's cloud provider. So reach out directly to aws or ... and provision sth called a load balancer.
This load balancer exists completely OUTSIDE of our cluster, it is a part of google cloud or ... .
This load balancer is gonna be used to take traffic from the outside world and direct it in to some pod inside of our cluster.
So the goal of a load balancer service is to tell cluster to reach out to it's provider, provision a load balancer(not a load balancer SERVICE!) with the goal of getting 
some traffic into a pod on our cluster.
A load balancer(not a load balancer SERVICE!) by itself is not useful for us right now. Instead, what we really want to do, is we want to get traffic 
distributed to a SET of different pods and we want to have some routing rules inside of sth to decide where to send this traffic to?

Note: Our react app doesn't want to uderstand the different between for example the query pod, comments pod and posts pod. We don't want to have to encode that info
inside the react. Instead, what would be great, is if we can just say: Make a req to this route and if there was sth around our cluster to figure out
what pod that req should be forward to?
So the load balancer just by itself, is not really doing the full thing for us, it's not doing everything we need and that is where the idea of an ingress or ingress
controller is gonna come.
Ingress or a ingress controller is a pod that has a set of routing rules inside of it. It's gonna work alongside that load balancer service.

Look at 88-028-2.
In the diagram we have our cluster which is the blue box and is being hosted with some cloud provider and we want to get some outside traffic into thos e
set of different pods.
A req is gonna coming in still to a load balancer that has been provisioned with our cloud provider. But now, that load balancer is gonna send that req on to the
ingress controller thing and that ingress controller thing is gonna have a set of routing rules inside of it. The rules means ingress would look at the path of req and 
then decide upon that path, whether to send the req on to that pod(actually send it to some cluster IP service that is send req onto the pod) or the other one.
(We're not showing the cluster IP services in the pod.)
That is the relationship between load balancer and ingress controller. The load balancer is just about getting traffic into our cluster. The ingress thing is about
routing rules or having some routing config that's gonna send req off to the appropriate cluster IP service which then gonna forward it to the related pod.

Now we're gonna look at a very specific implementation of an ingress controller that we're gonna use inside our app.

## 89-029 Update on Ingress Nginx Mandatory Commands:
In the upcoming lecture, we will be installing Ingress Nginx. In the video, it is shown that there is a required mandatory command that
needed to be run for all providers. This has since been removed, so, the provider-specific commands (Docker Desktop, Minikube, etc) are all that is required.

https://kubernetes.github.io/ingress-nginx/deploy/#provider-specific-steps

## 90-030 Installing Ingress-Nginx:
Ingress nginx: A project that will create a load balancer service + an ingress for us.
We're using ingress nginx. There is another project that does the same thing with an almost identical name which is named kubernetes-ingress.
These are very different projects. In the docs, go to deployment > installation guide.

Note: The kubectl apply -f <yaml config file>, is gonna take a config file and throw it into a cluster. You can also use this command to isntall packages for
your cluster. But first, take a look at that config file you want to apply to your cluster and understand it.

Note: A deployment is sth that creates and manages a set of pods.

Run:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
to use nginx-ingress.

Don't use the 'for development' section on docs because that's for developing the ingress-nginx. 

Now we've added load balancer and ingress controller to our cluster but not yet set up any routing rules or anything like that around our ingress controller.

## 91-031 Ingress v1 API Required Update:
When running kubectl apply in the upcoming lecture, you may encounter a warning or error about the v1beta1 API version that is being used.

The v1 Ingress API is now required as of Kubernetes v1.22 and the v1beta1 will no longer work.

Only a few very minor changes are needed:

https://kubernetes.io/docs/concepts/services-networking/ingress/

Notably, a pathType needs to be added, and how we specify the backend service name and port has changed:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: ingress-srv
annotations:
kubernetes.io/ingress.class: nginx
spec:
rules:
- host: posts.com
http:
paths:
  - path: /posts
pathType: Prefix
backend:
  service:
     name: posts-clusterip-srv
     port:
     number: 4000
  We will include a separate v1 Ingress manifest attached to each appropriate lecture throughout the course so that students can refer to the changes.

## 92-032 Writing Ingress Config Files:
Look at 92-032-1.

We have now created a ingress controller through ingress nginx, inside of our cluster. Now we need to teach this thing a couple of routing rules and tell it
how to take incoming reqs and send them off to some appropriate pods(actually the cluster IP services). We do that by writing a config file that's gonna contain some
router rules. Then we feed that into our cluster where it will be automatically discovered by the ingress controller. The ingress controller is then gonna 
update it's own internal set of routing rules to match the ones we just gave it.

So create a file named ingress-srv.yaml in k8s directory.

annotaitons key is important in there. 
This entire section of annotations:
annotations:
    kubernetes.io/ingress.class: nginx
is what's gonna help the ingress controller, understand that we're trying to feed it some routing rules. So the ingress controller is gonna continously scan all the
different objects or all the different config files that we're throwing into our cluster and it's gonna try to find one that has this exact annotation that we specified.
When it finds that, the ingress controller is gonna say: Oh, this thing must have some routing rules for me.

The `rules` inside `spec`, is gonna have all the different routing RULES that we want to apply to teach the ingress controller how to take incoming
traffic and route them up toward different parts. `rules` is gonna be an array.

**Note:** To designate an array entry in yaml, we use a dash at the beginning.

The first routing rule that we're gonna set up, is gonna take incoming traffic and send it off to our posts service. Now take a look at posts micorservice code.
There, we expect to get traffic on the route of '/posts'. So we're gonna make sure that if any req is coming into our app with a path of '/posts',
we're gonna send it off to our posts microservice. To do so, we're not gonna send it directly to the posts pod or anything like that, instead, we're gonna send it to the
posts cluster IP service, because remember these cluster IP services are how we communicate between things INSIDE of our cluster.
To write that we say:
path: /posts
backend: 
    serviceName: posts-clusterip-srv
    servicePort: 4000
BUTTTT!!!! The new syntax is:
path: /posts
  pathType: Prefix
  backend:
     service:
        name: posts-clusterip-srv
        port:
           number: 4000 (the port inside the app that we're listening on)

Now apply that new config file by going to the directory where that config file exists and run: `k apply -f <name of config file for ingress>` and you will
see sth like 'created'.

Now, in theory we have updated configuration of that ingress controller. But how do we test this out? (It comes back to the host property that we specified earlier.) 

## 93-033 Important Note About Port 80:
In the upcoming lecture, we will be editing our hosts file so that we can access posts.com/posts in our browser. If you are unable to access the
application you may have something already running on port 80, which is the default port for the ingress.

You'll need to identify what is using this port and shut it down. Some students have even had applications from other courses or personal projects still running.
For Windows Pro users, both SQL Server Reporting Services (MSSQLSERVER) and the World Wide Web Publishing Service / IIS Server have been the most common services causing a conflict.

To determine what might be using this port, run:

macOS / Linux

sudo lsof -i tcp:80

Windows:

netstat -aon | findstr :80

Minikube users on Windows and macOS should make sure that they aren't using the docker driver which is not compatible with an ingress as noted here:

https://www.udemy.com/course/microservices-with-node-js-and-react/learn/lecture/23145358#questions

## 94-034 Hosts File Tweak:
We just wrote our first ingress config file. But what is `hosts: posts.com` there?
When we make use of k8s, we can absolutely just hose one single app at one single domain. But with k8s, we can host a ton of infrastructure. We're not
necessarily limited to just hosting one singular app. 
**Important:** So we could host many different apps at many different domains inside of a single k8s cluster. Look at 94-034-1.
We could have some app tied to my-app.com, another totally different app at post-tracker.com . 
The infrastructure for all these different applications at these different domains can be hosted inside of one single k8s cluster. Ingress nginx is set up, assuming that
you might be hosting many different apps at different domains. So that's what that `host` property is all about. We're saying that the config there we're 
about to write, in the section where we have:
- host: posts.com
  http:
     paths:
        - path: /posts
        pathType: Prefix
        backend:
           service:
           name: posts-clusterip-srv
           port:
              number: 4000
is all tied to an app hosted at the value of host property which in this case is posts.com .
Now there's kind of a good side and a bad side to this. The bad side is that in development environment, we're used to accessing all of our different running
servers at localhost. So how does this kind of routing rule or this idea of having some specific domain like posts.com really stack up with
ingress nginx?
In dev env, what we have to do is trick our local machine into thinking that posts.com is equivalent to localhost.
In other words whenever we try to connect to posts.com , we're gonna trick our computer to connecting to our local machine rather than the real posts.com that I'm sure it 
exists out there somewhere online. 

To trick our local machine into connecting to localhost whenever we go to posts.com , we're gonna make a configuration change on our computer to our hosts file.
Look at 94-034-2.
Your hosts file is where we can set up a series of additional little routing rules and we can say anytime they tried to go to posts.com , instead just 
connect to our local machine. Again, this is going to somehow trick or make ingress nginx think that we're coming to it from posts.com and it's going to 
apply all the routing rules that are shown in the config file of ingress nginx.

Add this to the bery bottom of hosts file:
`127.0.0.1 posts.com`
If you're on minikube, instead write this:
`<result of minikube ip command> posts.com`

So now whenever we try to connect to posts.com , your OS is gonna say: Oh, rather than going to the real posts.com somewhere out on the internet, I'm going to
instead connect to the local machine(127.0.0.1) or me, which is at 127.0.0.1 .
If you see permission warning, in windows, open your editor program in administrator mode, if you're on macos just put in your password in the editor.

Now inside of the browser or postman or ... , if we go to posts.com , we're gonna be actually make a req to 127.0.0.1 , that req is gonna go into ingress-srv ,
then ingress-nginx is gonna think that we're trying to visit posts.com and so it's gonna apply the routing rules that we wrote for posts.com in the related
config file.

Now test this by going to posts.com/post in the browser which should behind the scenes, make a req to our posts cluster IP service which actually makes a req
to our posts pod and we know if we make a GET req to /post that should send us back all the different posts we have created inside the service.

So we're able to access some pod and most importantly, we didn't have to reach out directly to that pod. In other words, we didn't have to type in some url
that says: go to my posts pod, instead, we put in just a very normal URL and behind the scenes, ingress-nginx is gonna take that req and route if off to the 
appropriate service and that service(cluster IP service) in turn is gonna route it off to the appropriate pod.  

Now we need to go to all of our different services and we need to add in some routing rules to that ingress-nginx config file and we also need to 
create a deployment for our react app. Look at 94-034-3


## 95-035 Important Note to Add Environment Variable:
Please don't skip this!  You must make a small change to get a step shown in the next video to work correctly!

The next video is going to show the deployment of the React app to our Kubernetes cluster.  The React app will be running in a Docker container.

Unfortunately, create-react-app currently has a bug that prevents it from running correctly in a docker container.  Create-react-app does 
an open issue tracking this: https://github.com/facebook/create-react-app/issues/8688

To solve this, we have to make a small update to the Dockerfile in the client folder.  Find the Dockerfile in the client folder and make the following change:

FROM node:alpine

### Add the following line
ENV CI=true

WORKDIR /app
COPY package.json ./
RUN npm install
COPY ./ ./

CMD ["npm", "start"]
Then save the file.  That's it!  Continue on to the next video.

## 96-036 Deploying the React App:
Whatever value for host key you have in the ingress-srv.yaml , you need to add it to hosts file to have sth like: 127.0.0.1 <value of host key>, if you want
the machine to go to the localhost instead of the real domain.

Modifying the hosts file is really just appropriate for a development env. Once we start to deploy that thing out online, we don't need to make any 
actual changes to any host file on remote server or anything like that, because we will be connecting to the real posts.com which would hopefully connect us 
over to our k8s cluster.

Now let's set up the react app dev server pod. We need to make sure that inside of that react dev server, our actual react code is now gonna make reqs to posts.com .
Because that's where in theory our k8s cluster is located. If we make a req to posts.com , our react app is gonna attempt to connect to that ingress nginx pod.
Look at 96-036-1 .
So in the react app code, anytime we made a req to localhost, we will change it to be posts.com instead.

Note: In react app, when we say: await axios.post(`http://posts.com/posts/${postId}/comments`, {content});
that req is not gonna go the ACTUAL posts.com on the internet, it's going to be to our local machine because of the config in hosts file.

Now all the code inside of our react app is gonna try to go to posts.com . Again, when we're running that react app on our local machine,
whenever we make that req, our local operating system is gonna take that posts.com and redirect the req to our local machine instead, where it will be handled by
k8s.

**Note:** Now that our react app is using the right url for sending req, we need to create an image of it. Then we will write a config file to make a delpoyment, so we can
actually host that inside of our k8s cluster. We need to not only make a deployment, we need to make sure we make a cluster IP service as well, so that
the ingress nginx controller can eventually direct traffic to that pod so we can serve out the html, css and js out of it. 
So go to that directory where the react app exists and build an image out of the react application:
```shell
docker build -t <docker id>/client . # client is the name of the image that we're gonna build
docker push <name of the image>
# Now write a config file to create a k8s deployment to host that image that we just put together and also create a cluster IP service to go along with it. 
```
We named the config file for the deployment object of our react app, client-depl.yaml .

Note: A create react application is hosted on port 3000.

After creating that config file for deployment of react application, apply that config file tok8s cluster by navigating to the folder where that config file exists:
```shell
k apply -f <name of the config file with extension>
```

Now that our react app is also running in the cluster, we need to set up all those routing rules. We need to make sure that anytime we get an incoming req to that ingress controller,
it understands where to route the req off to? (Look at 96-036-2)

## 97-037 Unique Route Paths:
We're gonna set up a routing rules for all the other microservices inside of our cluster.

Right now we only have a route mapping for /posts in ingress-srv.yaml which is:
- path: /posts
  backend: 
    serviceName: posts-clusterip-srv
    servicePort: 4000
which means in theory, we can only attempt to fetch or retrieve posts, because that is the only thing we're doing with our posts cluster ip service.
So we need to add in some additional paths to this thing to make sure we map up all the different incoming reqs that might come in.

All the different routes that we need to service, is shown in 97-037-1. In that diagram, we have some reqs coming in from our react app(red boxes). Those are all 
the different reqs we expect to receive with their HTTP methods.

Note: A GET to / , is attempting to LOAD UP THE REACT APPLICATION, so we need to make sure that gets sent to the react pod.

Bad news is we have two routes right now of /posts! If a req comes in with a method of POST, then that should go to the posts service, otherwise if it is
a GET, it should go to query service. The bad news is that our ingress nginx module can NOT do routing based upon the method of the req!
So the only thing we can really route on effectively, is a route of req(path) and not it's method. So we need to route based on 97-037-2.
Now the reason this is bad, is that if we have a req coming in to /posts , we have 2 pods that can receive a req with path of /posts.
We can no longer differentiate on the METHOD of the incoming req. So now we don't have enough info with these reqs to figure out whether or not one should
go to the posts pod or query.

To fix this, we need to make sure that we have some unique path that should go to posts and a unique path for one that should go to query.
In other words, we just need to change the name of one of these paths that is repeated.
Instead of /posts for posts pod, change it to /posts/create . Now we need to change 2 different images just to solve this issue. React image and the posts service image.
Because we're gonna have to edit those two code bases and then rebuild the images and push them back up to docker hub.

So in react app, make the url for sending a req to create a post, to /posts/creates . Why? Because we want to differentiate between a req
that is supposed to go to the posts pod vs the query pod.
Then in posts service, change app.posts('/posts') to app.posts('/posts/create') .

Now inside react root directory, run:
```shell
docker build -t <docker id>/client .
docker push <docker id>/client
kubectl rollout restart deployment <name of the deployment>
```

Question: Do we have to rebuild the image manually and push it up to docker hub every single time we make a change to our codebase?
Yeah, it is awkwared, we're gonna fix it quickly. We're gonna look at a tool that's gonna automate this change process for us.
**Note:** For now: Anytime we want to make a change to our code, we have to rebuild the image, push it and then update the related deployment manually and then
restart the deployment.
With that, we've got updated the images and restarted the related deployments based on new source code.
Now we need to write the routing configuration that we need.

Now we've got unique routes that we're going to access in each of our different pods.

## 98-038 Final Route Config:

## 99-039 Introducing Skaffold:

## 100-040 Skaffold Setup:

## 101-041 First Time Skaffold Startup:

## 102-042 A Few Notes on Skaffold:




