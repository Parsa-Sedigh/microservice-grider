# 05 - Architecture of Multi-Service Apps

## 103-001 Big Ticket Items:
### Lessons from App #1: 
- The big challenge in microservices is data
- Different ways to share data between services. We are going to focus on async communication.
- Async communication focuses on communicating changes using `events` sent to an event bus it is then up to that event bus to take that event and send it around to all the
  different services inside the app and each of those services can react to that event in some way.
- Async communication encourages each service to be 100% self-sufficient. Relatively easy to handle temporary downtime or new service creation
- Docker makes it easier to package up services
- Kubernetes is a pain to setup, but makes it really easy to deploy + scale services

About #1:
If we have anything beyond just a blog, working with data between multiple different services is gonna start to become challenging.

About #2:
We learned that there are more than one way to share data between services. We spoke about the differences between sync and async communication. 
There really is a sliding scale on async vs sync communication. So we can have fully sync, we can have fully async, we can also have a combination of the two.
We're gonna focus on async communication between services on this course.
The reason for this, is that sync communication is generally a bit easier to implement, even though it kina has some downsides in actual production environment.

About #3:
We take these events which can be an object, array, string, number or ...  and we send them to an event-bus. It is then up to that event-bus to take that 
event and send it around to all the different services inside of our app and each of those services can react to that event in some way.
One of the nice things about async communication is that every service generally ends up being rather self-sufficient. As I mentioned, no, these things are not
truly self-sufficient, because they are relying upon events coming in from outside sources, but if any other service starts to go down, we can generally still have the
rest of our app work. Because we do not have strict dependencies between our different services.

About #4:
We saw that it's easy to use docker to package up all those services.

About #5:
K8s is gonna make it a lot easier to deploy our microservices and scale them up over time.

### painful things from app #1:
Also these are the things that we're gonna handle differently in second app:

- lots of duplicated code
- really hard to picture the flow of events between services
- really hard to remember what properties an event should have
- really hard to test some event flows
- my machine is getting laggy running k8s and everything else ...
- what if someone created a comment after editing 5 others after editing a post while balancing on a tight rope ... 


About #3:
We were working between multiple different projects and we would have inside of one project, we would emit an event and then we would attempt to receive that in another service.
So we were essentially sharing information across a boundary and we didn't have any good documentation in place or anything like that to understand the different properties
or even the names of those properties or even the types of data that those properties were referring to, in our events.

About #4:
For example, that moderation thing, to test out moderation, we first had to create a post. We then had to create a comment. We had to then(currently) refresh the page.
We then had to go through the other possibilities. So a comment with some bad words or flag words, or without those, maybe a different spelling or the capitial versions of 
those words. So it's hard to test out the event flows. We had to think about edge cases and ... .

About #5:
What if someone create a comment, after editing 5 others and then editing another post and then while at the same time creating a post?
Maybe there is some kind of like magic series of events, where if we emitted some series of events very quickly inside of our app, we might
actually be able to break our data model!
For example maybe some events would flow around our app out of order. Imagine a case in which maybe someone created a post and then simultaneously in the same
microsecond, created a comment attached to that post. We were making some really big assumptions inside this app about the ORDER of events flowing around.
What would happen if someone created a post and a comment associated with that at the same time? and then maybe for some reason, the 'comment creating' event
showed up instead of our comments service BEFORE 'post creation' event?
Our comments service was assuming that there was always going to be a post inside of it's little in-memory data structure to associate an incoming comment with.
So if we for some reason received a comment creating event BEFORE a post creation, our entire app would break.
So what if there is some really weird series of events, while some really crazy scenario is going on that might potentially break out our app?

painful issues of app #1:
duplicate code --------> build a central library as an npm module to share code between our different projects
hard to picture the flow of events between services ---> precissely define all of our events in this shared library
hard to remember properties of an event ---> write everyting in TS
hard to test some event flows ---> write tests for as much as possible/reasonable
machine is getting laggy running k8s and other stuff ---> run a k8s cluster in the cloud and develop on it almost as quickly as local
weird or out of order events ---> introduce a lot of code to handle concurrency issues

About #4 issue, we're gonna see how to run k8s cluster in the cloud in a development env. So this is not going to be a production style cluster, that is gonna be a 
cluster specifically intended to allow you to do development on a large k8s cluster in the casae that you might be running to many servies to really execute on
your own local machine. So you can kinda imagine that if you're working at a company that has dozens of different microservices, you might need to: 1) have
some kind of testing or development environment that works with all those different services and 2) maybe a dozen of services is too much to run on some old 
laptop. We solve that by creating a cluster in the cloud, a development cluster and we're gonna use the same kind of skaffold workflow with that remote cluster.

About #5: We're gonna have unending focus on data consistency and making sure that if events come into our app in some out of order, out of sequence, we're gonna have
some code or some idea of how to deal with that.

## 104-002 App Overview:
We're gonna build sth similar in nature to stubhub.com . 

There, in main page, we see a lot of different events and ... and we can select some time or data to go to that event, select it, then we prompted to select an 
actual ticket and those are the tickets that are for sale by other users. Select a ticket and then go to checkout(if you aren't logged-in, you need to login or signup first).

Imagine I'm trying to purchase a ticket for a very popular event. So maybe the instant that tickets get listed for sale, people just dogpile on it!
In other words, tons of people come to this website simultaneously and try to buy the exact same ticket. You probably want to make sure that nobody clicks on "go to checkout" and
enter their payment information for a ticket that has already been sold to some other user.  So we probably want to have some ability to say: "Hey, sorry, but someone else is 
in the process of purchasing these tickets" or sth similar. 

Let's look at a brief summary of some of the business rules around the app that we're going to put together.

So we're gonna build a ticketing app. We're not gonna have features like different dates or defined events or ... . We're gonna focus on the ticketing aspect of it. So we would have:

### overview of business logic inside the app:

- users can list(create) a ticket for an event(concert, sports) for sale
- other users can purchase this ticket(after it's been listed)
- any user who signs up to our application, can list tickets and purchase tickets
- when a user attempts to purchase a ticket(when user clicks on some purchase button), the ticket is 'locked' for 15 minutes. The user has 15 minutes to entier their payment info
- while locked, no other user can purchase the ticket. After 15 minutes, the ticket should 'unlock'. 
- ticket prices can be edited if they are not locked

About #2:
so no user is more special than another. Anyone can sell a ticket, anyone can purchase that ticket) and in our app, we're even gonna say you could
purchase your own tickets if you wanted to for some crazy reason.

About #3:
We're gonna take clicking on sth like "go to checkout" button, as a user
intending to buy that ticket and at that point of time, we're gonna lock the ticket and no other user should be able to purchase this ticket from out from underneath this
person. When we click on "go to checkout", we've not actually entered any payment information or anything like that, so a user still has to pay or enter his payment
info within 15 minutes, it is just expressing intent to purchase the ticket, that's gonna lock it for 15minutes. You can make it 30 seconds to test it.

About #4:
it's not even gonna be displayed to other users. So the instant someone locks the ticket, other users who are
searching for tickets, they just won't even see this one that has been locked.(This gonna be challenging, because it's gonna clearly demonstrate some big challenges around
async communication in microservices that can easily tell you why? Imagine the scenario in which some user out there lists a ticket for one hundred dollars and in some amount
of time later, another user clicks on purchase ticket. So like in one instant in time, a user attempts to purchase the ticket and the things should be locked and let's imagine
that at the SAME EXACT POINT IN TIME, the user who owns the ticket and who listed it, changes the price to 1000$. So at the same time, one person tries to lock it, at the same time,
someone else tries to change the price. We're gonna run into a ton of scenarios like that. We have to figure out how to handle these events in some precise, testable fashion.)

An order is when a user has locked the ticket they have created in order. Also, when a user has successfully purchased a ticket, that is in order as well.

## 105-003 Resource Types:
We're gonna design the app. We're gonna first begin by discussing the different pieces of data that we're gonna have to store, in order to implement this thing.

In total, we're gonna have 4 different types of resources:
Some table or collection, some storage of users. We're gonna also have: tickets, orders and charges.
For every ticket, we're gonna have a userId reference, which is gonna be a reference to the user who is trying to sell the tickets and a reference to some
order and that's gonna be the orderId field and that orderId reference is gonna be the kind of purchase attempt to actually get at this ticket.

An order represents the ATTEMPT to purchase a ticket. So the instant that a user clicks on that "purchase button", we're gonna create that order object. The order object
represents the intent to purchase the ticket. That expiresAt field represents the 15 minutes. As soon as the person actually enters in their credit card or what have you,
we will update the order status to Completed or AwaitingPayment.

Charge is gonna represent our ability to actually charge some person's credit card and get some money out of them. On the charge object, we'll have a reference
to the order that users trying to pay for as orderId.
We also have 2 mainainance fields or kind of administrative fields which are stripeId and stripeRefundId. stripeId is gonna be a reference to some object over in the 
stripe.js or the stripe world, that is gonna be us trying to actually bill someone's credit card.
We're also gonna have the ability to refund tickets as well. So we're gonna have sth called stripeRefundId too.

## 106-004 Service Types:
We're gonna look at different services that we're gonna design to manage each of those resources. 

In total, we're gonna end up with 5 different services:
- auth: everything related to user signup/signin/signout
- tickets: ticket creation/editing. Knows whether a ticket can be updated. It's gonna to know everyting about a given ticket.
- orders: order creation/editing . Very similar to tickets service.
- expiration: watches for orders to be created, cancels them after 15 minutes. It's gonna watch for anytime an order gets created. Anytime an order is created,
  it's going to attempt to cancel that order after 15 minutes has elapsed if a user has not already entered in some payment info, 
- payments: handles credit card payments. Cancels orders if payment fails, completes if payment succeeds. If a user enters in some credit card info and 
  we successfully charge their card, we're gonna mark the accompanying order that is tied to that payment, as Succeeding. Otherwise, if for some reason, the 
  order is canceled or if the order times out, or ... , we're gonna make sure that we mark a order as failed and the accompanying payment as failed.

We're essentially creating a separate service to manage each type of resource(except the expiration service). Look at 106.04.1 .
Is this the best approach?
Probably not. It really comes down to what kind of app you're trying to build, how many different types of resources you have inside of your app and the amount of 
business logic or special handling that is attached to every different type of resource.

Instead of creating a service for each type of resource, you might thinking about how you could create a service to handle each different **FEATURE** of your app?
For example a single microservice to handle ticket and order creation. A separate one to handle incoming payments and handle all aspects of a payment. For example
making sure that you mark a order as being InCompleted and handle that incoming payment. And you might have that expiration service tied into that tickets and 
ordering things, because they're all kinda tightly coupled together.
So it shouldn't be your default to say: "Oh, here's the type of resource, I'm gonna create a single service to manage it.", instead it comes down to your app.

But in this course, we're gonna create one service per type of resource.

## 107-005 Events and Architecture Design:
Let's list the different events we're going to create:
UserCreated, UserUpdated
OrderCreated, OrderCanceled, OrderExpired(whenever a order goes over that 15 minutes limit)
TicketCreated, TicketUpdated
ChargeCreated

Look at 107-005-1 . In this pic, we see the overall design of each service.

We're using redis for expiration service for a very particular reason.

Each of those different services are gonna be making use of a shared common library and we refer to this as common.

For an event bus, rather than building our own, we're gonna be using NATS streaming server(definitely not the same as network address translation which is in networking
world!!!)

## 108-006 Note on Typescript:

## 109-007 Auth Service Setup:
Look at 109-007-1 . In total, our auth service is gonna have 4 different routes assigned to it.

We use ts-node-dev which is a tool that we need to actually execute our project in a development env.

To generate a tsconfig file, run:
```shell
tsc --init
```

Note: The port which you listen on in your app's source code doesn't really make a different. So for example in express, the port which you specify 
in app.listen(<port>, () => {}); doesn't really make a difference once we start making use of k8s, because we're gonna start to create all those different 
services(a k8s service that is), that's gonna govern access to that express application like auth app that we're putting together.
So let's use 3000 as port of auth app(doesn't really make a difference).

We're going to EVENTUALLY run our express apps inside of k8s, but FOR NOW, let's run the express app like auth, with `npm start`.

## 110-008 Auth K8s Setup:
Anytime that we start to set up k8s, we need a couple of different things. First we need to make sure that we can build an image out of our service like
auth service. 

In order to build an image, remember, we have to write out a docker file. So create a Dockerfile inside auth directory.

We don't want to load up the node_modules folder into our container as it's being built. So create .dockerignore and add node_modules to it.

Now build the image of auth service. First stop the server of auth service(if it's running).
So run: `docker build -t <your docker id>/auth .` inside the directory where the related Dockerfile exists.

Now we want to write more config to get that built image loaded up into a k8s cluster.
To get that container running inside a k8s cluster, we're gonna create a deployment. 

**Note:** A deployment is gonna create a set of pods for us automatically and we can make sure that those pods are running our auth service.

To write a k8s config file, create a new folder called infra(if it doesn't exist yet), then there, create a new folder called k8s and there, create a new file
called auth-depl.yaml .

Inside the `spec` section of deployment config file for a service, we first designate the number of pods that we want to run for the related service. So specify
`replicas` for that. For example replica: 1 means we just want to run 1 copy of that image.

This:
```yaml
  selector:
    matchLabels:
      app: auth
```
is the step 1, in a little 2 step process.

The purpose of `selector`, is going to be to tell the deployment, how to find all the pods that it's going to create. 

**Note:** After setting up that selector, we set up the `template` which shows how to create each individual pod that that deployment is going to create.

The thing we write in `mathLabels` section of `selector`(which in case of auth service, that thing is: `app: auth`), is kinda matching up with the `labels` section that
we write for `metadata` of `template`.

The `spec` section in metadata, is going to tell the pod how to behave. There, we're gonna first designate all the containers that are going to be running inside that pod by
using the containers section and it can have multiple entries, so use a dash to indicate an array entry.
The `name` that you use for each container in that containers section is only important for logging purposes(so: `-name: auth` is only for logging purposes). 
The value for `image` is the name of the image that you just built for the related service.

Anytime we create a deployment, we usually are going to want to end up creating a service to go along with it. This is a k8s service, sth to give us access or 
allow us to get access to a pod. We COULD create a separate file inside of our k8s directory to house the service config file, but a lot of time, we're gonna 
end up with a one to one ratio between our deployments and the services. So we can just define a service inside the same file as the deployment config, so that all the 
stuff is put together in one single location.

So just use three dashes to separate those two configs.

**Note:** The `spec` section tells th service how to behave and there, the selector tell that service how to find the set of pods that that service is supposed to govern
access to(the service is gonna govern the access to those pods). We want to find all the pods with th elabel of `app: auth`, so as selector, use `app: auth` .

With `ports`, we list out all the different ports that we want to expose on that pod. For now we only have one port, so that `ports` section only has one entry.

For auth service, we set 3000 as port and targetPort because our auth service source code, we had it listening on port 3000.

Important: Notice that for `spec` section, we did not define a `type` for the service. Because we're relying upon the default. The default service whenever we create one,
is a cluster ip service. A cluster ip service is going to allow communication to the service from anything else running only inside of our cluster.

The goal of skaffold is to find all the different things that we want to throw into our cluster, build them up and then throw them in and also handle some live code reload for
any changes that we make to our project as well.

## 111-009 Adding Skaffold:
Let's setup the skaffold config file. Skaffold config file is gonna watch our infra directory, anytime we make a change to a config file in that infra directory,
it will automatically apply it to our cluster. It's also going to make sure that anytime we change any code inside of our auth directory, it will sync
all the files inside there with the appropriate running container inside of our cluster.

Now go to root project directory and create skaffold.yaml .

Note: In skaffold.yaml, the `deploy` section is gonna list out all the different config files we want to load into our cluster.
Remember to put a ./ at the start of entries to manifests, to say: from the current working directory(means ./) ... 

When we have:
```yaml
  local:
    push: false
```
it means that whenever we build an image, do not try to push it off to docker hub or anything like that, which is the default behavior.

**Note:** The artifacts in skaffold.yaml is gonna be all the things that are going to be built. 
So for now, in artifacts, as one entry, we're gonna list out the image that's going to be produced by that auth project.

The `context` key is the folder that contains all the code for that image.

The `sync` section is gonna tell skaffold, how to handle any files that change inside of that context.
There, `dest: .` means where to sync this file to, inside of our running container. The dot means basically just take wherever the file was found from(which in the
case of auth container, was found in src/**/*.ts) and throw it to the corresponding path(.) inside the container.

Now run skaffold to see if we can get our project up and running inside of our cluster. If you're running skaffold from the previous project,
do make sure that you close it before running it for this new project. Then go to the directory where the skaffold.yaml exist and run:
```shell
skaffold dev
```

If you get an error the first time you run skaffold, that's ok. Try killing the process with ctrl+c and re-run the command.

After running skaffold, looks like we succesfully build the image, we then started the deploy with skaffold, then a auth deployment(auth-depl) was created and
and then auth service(auth-srv) was created as well.
Then we can see the logs from our newly running container.

Sometimes we run into some big issues on detecting changes(actually skaffold detecting changes), made to projects that are running inside of a docker container. 
So if you don't see the immediate change, that's fine. 
Now let's see what you should do, if you do not see a change.

## 112-010 Note on Code Reloading:
Hi!

If you did not see your server restart after changing the index.ts file, do the following:

Open the package.json file in the auth directory

Find the start script

Update the start script to the following:

ts-node-dev --poll src/index.ts

## 113-011 Ingress v1 API Required Update:
When running skaffold dev in the upcoming lecture, you may encounter a warning or error about the v1beta1 API version that is being used.

The v1 Ingress API is now required as of Kubernetes v1.22 and the v1beta1 will no longer work.

Only a few very minor changes are needed:

https://kubernetes.io/docs/concepts/services-networking/ingress/

Notably, a pathType needs to be added, and how we specify the backend service name and port has changed:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - host: ticketing.dev
      http:
        paths:
          - path: /api/users/?(.*)
            pathType: Prefix
            backend:
              service:
                name: auth-srv
                port:
                  number: 3000
```

We will include a separate v1 Ingress manifest attached to each appropriate lecture throughout the course so that students can refer to the changes.

## 114-012 Ingress-Nginx Setup:
Before we worry about setting up any additional tooling around testing, we should make sure that we can do some manual testing using sth like postman.
So let's set up a route handler to test it.

We need to access our running server. Specifically, our auth pod. Remember that to access anything inside of our cluster, we have to set up one of two things.
We really have two options! We can either set up one of those node port services or we can set up that ingress service.

Remember that ingress is all about having some routing rules tied to nginx. So anytime a req comes into our cluster, it'll be handled by that ingress service and 
they'll be routed off to the appropriate service within our cluster. So let's add some config fules to ingress nginx.

Between the last app and this one, we did not delete ingress nginx or anything like that out of our cluster. When we stopped our skaffold tool from the 
last app, we deleted all the deployments, services and pods that were listed inside of our last project. We did NOT delete anything specifically tied to ingress nginx
beyond our set of config rules. So in other words, you should not have to reinstall ingress nginx right now, unless you for some reason, reset your cluster between the 
last set application and this one, or manually deleted all the ingress nginx stuff or sth like that.

So we should only have to write out a new config file for ingress nginx and that config file will be loaded into our existing copy of the ingress nginx controller that is 
still running inside of our cluster.

If you did restart your cluster, to get nginx reinstalled, all you have to do is to go Deployment page of ingress nginx and run the _mandatory command_ 
and then run either _docker for mac_ (it's also the docker for windows option) or the minikube command.

Now let's create a ingress config file. So create a file named ingress-srv.yaml and there, we write some config to tell the ingress nginx controller, how to
handle incoming reqs.

**Note**: In ingress config file, `nginx.ingress.kubernetes.io/use-regex: 'true'` is telling nginx that it should expect some of our different paths that we're gonna 
list, are gonna have regular expressions inside them.

**Note**: The host in specs of ingress config file, is gonna be a kind of pretend domain name. So we're gonna put in some made-up domain name that we're gonna be 
able to connect to, only from our local machine. So whatever domain name that we're gonna enter right there, we're gonna make sure we also
go over and edit our hosts file as well to include that domain name. So that domain name is gonna only work on our local machine.

```yaml
- path: /api/users/?(.*) # /api/users/<anything>
            pathType: Prefix
            backend:
              service:
                name: auth-srv
                port:
                  number: 3000
```
means anytime someone makes a req to our cluster and the req has a path of /api/users/<anything>, then we're gonna send that req on to that `backend`. In this case,
that backend is the service that has a name of auth-srv and on port 3000.

Now we should see: `ingress-service was configured`.

Now we need to update the hosts file on our local machine to make sure that anytime we make a req to ticketing.dev, it redirects that req to our local machine or 
it stays on our local machine and not the internet!

## 115-013 Hosts File and Security Warning:
Open hosts file(in administartor mode) which is on /etc/hosts on mac. Look at 115-013-1 .

After changing hosts file, to test this out, we should be able to navigate to ticketing.dev inside of our browser and when we go there, that's just gonna
reach out to our ingress nginx server. We don't want to reach to the server directly, we want to go to the route handler we put together, so we 
specifically want to go to a route of ticketing.dev/api/users/currentuser and that's how we can access that express route handler.

Now when you come to that ticketing.dev/api/users/currentuser, you might see an error message that says: Your connection is not private. This is because of some
of the default behavior inside of ingress nginx. You see by default, ingress nginx is a web server that's going to try to use an https connection. Unfortunately, 
by default, it uses what is called a self signed certificate. Chrome does not trust servers that use self sign certificates. As a matter of fact, if you
click on that red https in url and click on certificate, it's gonna say: kubernetes ingress controller fake certificate.
Usually, we could get around this message by click on advanced and clicking on that link, but unfortunately, kind of a double unfortunate thing!!!,
the ingress server uses some configuration options that are going to prevent you from trying to circumvent that warning screen.

So right now it seems like we just cannot reach out to that express route handler at all.(This error only affect us in dev env and it's sth our users are never
gonna see once we properly configure all this stuff).

**Note:** To get that message to go away, we're going to type: `thisisunsafe` into that tab. You don't have to select any inputs or anything like that.
You're just going to go over that tab, click anywhere inside of it and then type: `thisisunsafe`.



