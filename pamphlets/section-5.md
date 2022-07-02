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

## 105-003 Resource Types:

