## 06 - Leveraging a Cloud Environment for Development

## 116-001 Big Ticket Items:
We can now access our auth service from browser.

Look at 116-001-1.

In this section, we're gonna learn how to set up our development env to run through google cloud. The reason we're gonna do this, is that 
later on, we're gonna have many different pods running inside of our local cluster. If you're on an older computer, you might eventually start to exprience
a lot of lag or even crashes, because of the amount of code that we are executing.

We're gonna learn how to get around that, by running all of our code on google cloud instead of on our local machine.
 
To make use of google cloud, you must have a credit card. Look at 116-001-2 . You probably are not gonna have to pay any money, because we're gonna use a 
relatively small amount of resources on google cloud. So we should be fitting into the free tier, but that is not a guarantee.

## 117-002 Remote Dev with Skaffold:
Let's first get started with a review on how we're running k8s on our local computer right now and how we're gonna run it through google cloud?
Look at 117-002-1 which shows what's going on on our machine at present.

We're running a SINGLE k8s node and on there, we could possibly have many different pods. At present we probably only have 1 pod which is that auth pod,
but eventually we're gonna have many different pods.

We can open up our browser and access that node using the ingress nginx server we just configured.

Here's what's gonna happen once we start to move all this stuff to google cloud:
Look at 117-002-2.

Your computer is still going to be running, say, a browser or sth like that, trying to access our different pods and test out some code manually.
But our entire k8s cluster is gonna be running on a google cloud virual machine. So no longer are we going to be running k8s on our computer whatsover.
Instead, we're gonna leverage some remote computer. So we're gonna have a node somewhere out on the cloud that's running all of our different pods.

Now what skaffold is doing for us?
Skaffold is going to be the real tool that's going to allow us to set everything up on google cloud, easily.

Skaffold was actually developed by teams at google. That's why we're using google cloud.

Here's what's going on behind the scenes with skaffold:

Look at 117-002-3. We're going to add in some additional configuration to skaffold and tell it that we wanted to use some node hosted on google cloud as opposed to
a local node on our machine. Skaffold, is not only going to create all of our different services and deployments and ..., it is also in charge(very critically) of handling
changes that we're making to our project as well.

Recall that there are two types of changes that we can make to a file or a project when we're make of skaffold. 
1) We either make changes to a file that is synced or unsynced. We listed out that `sync` section in skaffold.yaml and this meant that anytime that we made a 
change to a file mathcing that pattern, just take that file and sync it with a corresponding part inside of our cluster. So in that scenario, we were
not rebuilding any image or anything like that. We were just taking the changed file and sticking it directly into a pod.

The diagram in 117-002-3 shows that scenario which is the case in which we make a change to a synced file. So on your computer, you're gonna be writing inside of 
your code editor and at some point in time you're going to change a synced file. So in our case, that's gonna be a ts file inside src directory of auth service.
Then whenever we save that file, skaffold will detect that change. Skaffold will then be configured to reach out to google cloud, to reach out to the node that 
is running this specific pod and take that changed file and stick it directly into that pod and that's how updates are going to be processed.

2) This scenario is where we change a NOT SYNCED file(a file that will not be automatically injected into a pod), is a bit complicated. Let's imagine that we make a 
   change to our package.json file(which is not matched by src/**/*.ts). When we change that file, it does not get automatically injected into a pod, because we 
   did not listed inside the sync section of related service in skaffold.yaml . Whenever we make a change to package.json , that is a sign that we want to 
   completely rebuild our auth image and then update some corresponding deployment inside of our cluster. 
   Look at 118-002-4 . In this scenario, we change an unsynced file. So on our computer, we're gonna open up our editor and make a change to a file that is not
   listed in the `sync` section. Skaffold will detect that and it's gonna add in an extra step. Skaffold is gonna reach out to a service that runs inside
   of google cloud called google cloud build. This service is all about building docker images. So skaffold is gonna take all of our source code for our project, 
   that's gonna upload it to google cloud build along with the Dockerfile for that specific project. Then on google cloud build, some builder process 
   build our image for us. The whole reason that the image is being build on google cloud, is so we decrese the number of resources that are being consumed on our 
   local machine. In addition, building the image is gonna be a lot faster because google cloud servers have extremely, fast internet connections and later on,
   we're gonna have a couple of projects or sub projects inside of our project that have very large numbers of dependencies, maybe not very
   large numbers, but we're gonna have deps that require downloading sth like 200 or 300 megabytes. So if you're on a slow internet connection, it might take a long time to
   re-download all those deps any time that we have to rebuild an image(because of changes to a non synced file like package.json).
   By making use of google cloud build, you're going to be able to install all those deps in a matter of seconds. So google cloud build is in charge of 
   rebuilding all of our images.
   Once the image is build, skaffold will then reach out to the corresponding deployment on our node and tell that deployment that there is a new image available. 
   The deployment will then restart all of our pods using the latest updated image.

We need to create a new project, we're gonna set up a k8s cluster on google cloud. We're gonna configure google cloud build. We're gonna configure skaffold to use 
google cloud build and that remote cluster as well.

## 118-003 Free Google Cloud Credits:

## 119-004 Google Cloud Initial Setup:
If you already have a google cloud account, you might want to create a brand new account, just you can get those 300 dollars in free credits to guarantee 
that you will not have to spend anything in this course.

Click on "new project" and name it "ticketing-dev".

## 120-005 Kubernetes Cluster Creation:
Create a new k8s cluster inside the newly created google cloud project. Go to `kubernetes engine`>`clusters`. Change the name of cluster to ticketing-dev.
Location type setting should be zonal and for zone, make sure that the zone that is elected is somewhere near you.

Master version is the version of k8s that we're gonna use. The versions that are listed, are usually pretty laggy. In other words, when a new version of 
k8s comes out, you will not see it appear in that list for quite a while. We want to make sure that we have at least a version at the 1.15... series.
You can select the latest version.

Then on left hand side, go to node pools. Then go to size and by default we get 3 nodes.

Remember each of those different nodes is essentially an env where our pods, our deployments, our services are going to be executed.

We also need to configure exactly what each of those nodes are? So on left side, select nodes. In machine configuration, by default they're selecting a fairly 
standard virtual machine to run all your different pods, this is a bit more powerful than we need and you actually end up paying for it as well!!!

So we can select `g1-small`.

Now click on create at bottom left hand side. That's gonna create a new cluster of 3 separate nodes.

## 121-006 Kubectl Contexts:
How do we somehow connect to the cluster that we just created in google cloud?
To access different clusters, we're always gonna end up using the same kubectl command. Behind the scenes, there are some configuration options that are
being applied to kubectl. When we change those config options, is gonna change which cluster we're connecting to. 

Look at 121-006-1.

Behind the scenes kubectl uses sth called context. You can think of it as some different connection settings. They list out some authorization credentials, 
some users, some ip addresses, a lot of different info to tell kubectl how to connect that to different clusters that exist in the world.

Right now, we're connecting to our local cluster through a context that was created when we first installed docker for windows or mac on our machine.
As a matter of fact, on docker icon>kubernetes , you can see sth that says context and our current context is docker-desktop.
So that context has some connection settings inside of it to tell us how to connect to our local cluster. We need to add in a second context, a 
new one that's gonna tell kubectl how to connect to the cluster that we just created in google cloud.

There's two ways we can do this. We can either use the google cloud dashboard and go to some menus to find some little config that we would copy paste into a 
file on our local machine. THat would be one way to add the context.

Another way which is easier, is to instead install a tool called google cloud sdk which is a command line tool that we can use to interface with google cloud automatically.
Google cloud sdk can create different contexts for us automatically and update kubectl on our local machine and teach it how to connect 
to the clusters that we're creating in google cloud.

So install google cloud sdk to automatically manage these context things for us.
Just remember you do not have to initialize the SDK just yet. We're gonna do that in the next vid(do not run gcloud init command yet).


## 122-007 Initializing the GCloud SDK:
----------------------------------------TODO----------------------------------------
Need a credit card to sign up to gcloud and continue this...

## 123-008 Installing the GCloud Context:

## 124-009 Updating the Skaffold Config:

## 125-010 More Skaffold Updates:

## 126-011 Creating a Load Balancer:

## 127-012 Final Config and Test:


