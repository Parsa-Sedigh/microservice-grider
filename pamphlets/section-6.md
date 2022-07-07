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



