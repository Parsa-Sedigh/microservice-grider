# A Mini-Microservices App

11-001 App Overview:
For every different unique resource that we have in our app, we're gonna create a separate service.
In our app we have two different types of resources: posts and comments. So we have two different resources. So two services.

Now the first thing we wanna do is picture the different goals or responsibility of each service.
With a post service, we want the ability to create a post and to list all posts and for comments we want the ability to create a comment
and list all comments.

There's gonna be a dependency of sorts between creating a comment and having some knowledge of what posts exist. So we're gonna apply one of those
async or sync communication patterns to make sure users can only create comments that are tired to a specific post. Also for listing comments, we wanna
show the comments that are just associated with a particular post.

12-002 Project Setup:
Create a new react app
create an express-based project for posts service and another one for comments service.
Create a directory for your whole project which contains multiple services. Then use create-react-app to initialize a react project and call it `client`.
Then create two services by creating a new directory for each and name them posts and comments. Then run npm init -y and then install these packages 
for EACH of those services: express cors axios nodemon

13-003 Posts Service Creation:
We're not gonna have any DB for this project, so we store data in memory. The downside is that anytime that we restart a service, we're gonna lose all
of our data.

We need to add in a body parser to make sure that whenever user sends us some JSON data in the body of req, it gets parsed and it shows up appropirately inside of a 
request handler.

14-004 Testing the Posts Service:
When sending reqs to our services, in header of the req you need to specify: Content-Type and it's value should be: application/json .

If we make any changes to our code inside of our project, that's gonna restart our server and because we're storing all of our posts in memory,
all of our posts will be automatically dumped!

Note: You don't need to install body-parser anymore, it's built-in but we do need to use: app.use(express.json()); for parsing req's json body.

15-005 Implementing a Comments Service:
First find the particular goals of a service you're gonna implement.

The properties of commentsByPostId object would be the ids of a post that has been created in our app. Then for each of those keys, we're gonna have an 
array of comments.

16-006 Quick Comments Test:

17-007 Note on the React App:
18-008 Suggestion Regarding a Default Export Warning:

19-009 React Project Setup:

20-010 Building Post Submission:

21-011 Handling CORS Errors:
Cores req are an issue any time that we are looking at some domain or domain with a port or domain with a subdomain that is different than the url or 
the domain that we're trying to make a req to. In our case, we're at localhost:3000 and we're trying to make a POST req to localhost:4000 . So the CORS
req policy comes into effect. The browser is gonna automatically reject or prevent any req from being issued from localhost:3000 over to localhost:4000 unless
localhost:4000 (our server) provides a very specific set of headers. 
In order to get this error go away, there's nothing we could do inside of our browser. There's no additional config we could add to our react app. we have to
do some additional config on our server itself.

So do this config on both post service and comments service. For both, install cors package. It's gonna resolve this error by setting a header on specific 
responses that are going to go back to the browser.

In the later project, we're not gonna have to worry about CORS error so much because there's a couple of architectural changes we're gonna add so that we will not
end up making reqs from one URL inside browser to some DIFFERENT URL where our actual APIs hosted. We're gonna make a couple of changes that they're
co-located so to speak.

22-012 Fetching and Rendering Posts:

23-013 Creating Comments:

24-014 Displaying Comments:
Currently we're making one request for EVERY post we have fetched to get the comments of that post.

25-015 Completed React App:
26-016 Request Minimization Strategies:
We want to make one single req and get all of our posts and all the associated comments for those posts as well.
If you try to do this in a sort of monolith architecture, this would be easy. We can say if we make a req to /posts?comments=true we want to get a list
of our posts with all of the relevant comments embedded in those posts as well.
But how we're gonna solve it with microservices?
Right now, we only have the ability to make a req to either the posts service or the comments service. 
There are two possible solutions and the pros and cons of both these solutions are gonna be similar to those methods of async and sync communication between
services.

The solution based on sync communication:
We're gonna make a GET req to posts service and then to make sure we get all the relevant comments embedded, we could add some code to our post service
to reach out automatically to our comments service and say: give me all the comments that you have associated with these post ids. The comments service
would then reply with all the relevant comments, then the post service would take those comments, assemble them all together with the relevant posts and 
then send the entire bundle back over to the browser. This solution relies on sync communication.
Cons of this approach are identical to the downsides of sync communication.

# 27-017 An Async Solution:
The goal of the event broker is to receive notifications from different services, take those notifications or those events and route them off to 
all the other services that we're running.

We're gonna introduce the idea of a query service.
The goal of query service is to listen to anytime that a post is created or a comment is created which anytime these events happen, they're gonna emit
an event. The query service will then listen for those events and it's gonna take all the different posts and comments that get created and
assembles all of the posts + comments into an efficient data structure that can solve our root problem right now which is to make sure that 
we reduce all these number of reqs down to just one single req.
So again the goal of this query service is to take all the different posts, all the different comments and just serve them up to the browser in one simple req.

Pros and cons of this approach:
Pros: 
- The query service that we're gonna create, doesn't really have any direct deps on other services. Yes, it does rely upon some incoming events that are
  being issued by those services, but if those services go down for any reason, the query service is still gonna work as expected. 
- The query service is gonna be fasat compared to that more sync style of communication. The reason for this is that we're not making any req between our 
  different services anymore. If someone wants to get a list of all the different posts and comments associated with them, it can be ONE req.
Downsides:
- data duplication
- harder to understand

28-018 Common Questions Around Async Events:
1) You might be thinking that if we have two types of resources, so in our case posts and comments, if we want to serve up that information effectively,
we have to create a third service?
No. We're not always going to be creating extra services just for the sole purpose of joining data together. In reality, if we were building this sort of 
blog app, this blog app will be part of a larger scheme of resources or a larger app. In reality, we would probably put this idea of posts and
comments together in the same service, so we could join that data together at the code level rather than at a service level by creating a third service.
So what we're doing now, is solely for learning purposes. So we're not advocating that you create an extra service for every last point of data that you need!
2) The fact that we can make these things independent and keep the vast majority of our app running even if some part of it goes down, that's a huge driving force
in why we're learning about microservices.

Now let's implement a new query service. Then set up the even bus and then have all of these services communicating with each other via events.

# 29-019 Event Bus Overview:
We're gonna use even bus to communicate events for one service over to another.

##Event Bus:
- many differnet implementations. RabbitMQ, kafka, NATS ... 
- receive events(some piece of information) publishes them to listeners
- many different subtle features that make async communication way easier or harder
- we're gonna build our own even bus using express. It will not implement vast majority of features a normal bus has.

Event is some piece of information. It can be json, raw bytes of data or string.

In the larger app we're gonna work, we will pick an event bus that's fast but also some downsides.

We're gonna add a new route to our services and in these routes, we're gonna watch for a POST req to a path of /events.

Whenever a service needs to emit an event, it's gonna make a POST req off to the event bus which is gonna be an express app.
Essentially we're doing a POST req from one service to another. That event bus is gonna receive that event and it's gonna take the event and make a series of 
POST reqs with it. So it's gonna take that event data and send it off with a POST req to localhost:4000/events which is the posts service, also an identical
request off to localhost:4001/events for comments service and ... .

So anytime one of our services makes a POST req over to event bus, event bus is just gonna take that exact information and throw it out to everything
else that we have running inside our app. EVEN THE SAME SERVICE THAT EMITTED THE EVENT ORIGINALLY.
So we're sending an event over to the event bus then it's gonna echo that event back to all of our services.

# 30-020 Important Note about Node v15 and Unhandled Promise Rejections:
Create a new folder for event-bus. Then run npm init -y there.

# 31-021 A Basic Event Bus Implementation:

# 32-022 Emitting Events:

# 33-023 Emitting Comment Creation Events

# 34-024 Receiving Events:

# 35-025 Creating the Data Query Service:
Our query service is going to care about events of type PostCreated and CommentCreated. Whenever it sees these events, it's gonna take the data contained 
inside the event and then assemble it into easy to access data structure.

'/events' route is the endpoint that's gonna receive events from our event-bus. 

# 36-026 Parsing Incoming Events:

# 37-027 Using the Query Service:
So we have introduced a brand new service into our app, it's consuming events from these other services and it's using those events to populate some
internal store of data and we did al this to make sure that we can minimize the number of reqs required.

Also our query service doesn't have any dependencies(direct dependencies) on other services. So if the comments service and posts service are currently crashed,
the query service is ok and we can still serve up data to users of our app without any issue. So the query service is independent.

# 38-028 Adding a Simple Feature:

# 39-029 Issues with Comment Filtering:
option 1: moderation service communicates event creation to query service.
A comment has 3 states. It can be approved, rejected or pending-moderation.
Now what would happen if the moderation service took a long time to run? Maybe a human would decide on a comment to approve it or not.
In this case(moderation process is not instantaneous), the flow is like this:
A user is gonna submit a comment, comments service is gonna persist that comment and the comments service will emit an event and that event is gonna be sent over to 
the moderation service from event-bus, where it's gonna await actual moderation. Now at that point in time, we're essentially paused in this workflow. Nothing else is 
gonna happen anytime soon.
Now think about UX. When a user refreshes a page or in case of ajax, after some seconds, they expect to see the new comment listed there immediately or at least sth 
that says: hey, your comment is pending moderation or ... .

So with the approach we're discussing right now, we're saying that after a user refreshes their page, they're not gonna immediately see that comment from the query 
service right away. Because it's gonna take some time that the moderation service to process that comment and then tell the query service that there's this new comment.

Pros:

Cons: 
- there is some delay between a user submitted a comment and that comment actually being persisted by the query service. So this entire flow could
  potentially result ina user not seeing the info that they just submitted, easily.

This idea of a user making a change and not seeing that immediately reflected, that's sth that's gonna come up all the time in this idea of microservices.
But in this case, because of possibility of the moderation service being ran by an actual human, we're kind of exacerbating that problem.

We're not go with this option.

# 40-030 A Second Approach:
option 2: moderation updates status at both comments and query service.
We're still going to have a user submitted comment to the comments service. We're still going to have the comments service emit an event of sth like
'CommentCreated' and ... . Event bus is still gonna send that event off to the moderation service, but now we're going to also have it go off to the 
query service and actually be processed by the query service. As soon as the query service sees 'CommentCreated' we're going to have the query service
persist some info about that comment and critically, it's also gonna have a default status of sth like 'pending'. 
The query service is gonna instantly know about this comment, even if it takes some amount of time for the moderation service to process and actually moderate 
the given comment.

This option and also the previous option have a persistent issue:
The job of query service is all about presentation logic. It's about making a query or storing some data and serving that data up quickly to users.
So right now, it's really joining just two resources, posts and comments but in future it might join other resources. 
This query service might take many different resources and jam them all together.

So a question:
Does it make sense for this presentation service to understand how to process this very precise update to a comment, like a 'CommentModerated' event?
Does it make sense for the query service to understand how to process an event like this?
No. In real world commenting system we would have:
type: CommentModerated
type: CommentUpvoted
type: CommentDownvoted
type: CommentPromoted
type: CommentAnonymized
type: CommentSearchable -> comment.searchable = true
type: CommentAdvertised

Each of these different precise updates to a comment has some very precise business logic associated with it to update the definition of that comment.
So with this approach, the query service needs to understand how to handle this multitude of different possible updates that can be made to a 
comment and imagine that if there were other services like our query service, maybe sth to process weekly updates to popular blog posts(which is done in 
weekly update service) or maybe sth that functions as a recommendation service which receive the above events as well, if those had to store THE IDEA OF WHAT
A COMMENT IS as well, in theory they might also have to handle these events. 

So we start to get into this scenario where every possible way of changing a comment has to be handled by a ton of different services.
So this approach is also not good.

# 41-031 How to Handle Resource Updates:
So the issue with the second approach is that we're allowing our query service to have a very deep understanding of what this 'CommentModerated' event is.
To solve this, we're gonna have the CommentModerated be instead processed by the comments service.
The comments service is the service that is in charge of exactly what a comment is and knows all the very precise business rules, all the business logic, 
everything around what a comment is, how to update it and how to deal with is in some way.
So comments service is responsible for updating a comment and deal with the mentioned types of events for updating a comment.
Whenever the comments service updates a comment by handling one of these very specialized events, we're going to have emit one single, very generic
event called 'CommentUpdated'. Whereas all the events that I mentioned above like CommentModerated, meant that we wanted to update a comment in some specific
way, the CommentUpdated event says: Hey, there's a new comment right here(in the data of event) or a new version of this comment, don't worry about what 
the update was. Don't worry about trying to interpret it, don't worry about trying to run some specialized business logic around it, just take these updated 
attributes and store them if you need them.

The service that is in charge of a basic resource, will be in charge of these very specialized updates and anytime those specialized updates occur,
that service will emit a very generic event update.

option 3 - query service only listens for update events. 
We will have two events: One is gonna be very specialized in nature like CommentModerated event, it's gonna be processed by the comments service and then 
as soon as the comments service updates that comment, it's gonna emit that generic event(CommentUpdated).

# 42-032 Creating the Moderation Service:
Create the moderation service. We don't need to install cors for this service because we're not gonna have our frontend app make any DIRECT req to this service.

# 43-033 Adding Comment Moderation:

# 44-034 Reminder about Node v15 and Error Catching:

# 45-035 Handling Moderation:

# 46-036 Updating Comment Content:
CommentModerated event is emitted when the moderation service moderates a comment.

# 47-037 A Quick Test:

# 48-038 Rendering Comments by Status:
To simulate the moderation process takes time, shut down the moderation service.

There's a problem in our app. Since we turned off the moderation service, if we start up the moderation service again, in the time that the 
service was down, the event broker(bus) tried to send an event over while the service which was interested in that event was down. So now we're in a scenario 
where we had a temporary interruption to our moderation service but we lost events in that window and so now our entire app is kind of out of sync.
We don't have any way to kinda fast forward ang get those missing events back over to our moderation service.

#49-039 Dealing with Missing Events:
Other than the case that a service goes down for some time, what happens if we're using this kind of event style approach and we don't bring up or we don't 
even create a service until maybe sometime in future?
For example maybe we had been running some services for days or years and they have tons of data inside them and they've all received tons of events.
But maybe we only created a new service like one year down the line.
How do we get that new service into sync?
There are couple of options:
1) sync reqs. 
The instant we launch that new service, maybe we would have some code inside of the new service to make a direct network reqs over to the other
services that were running for some time and say give me your data. Once it gets that data, it can make another req to other service that were running for
some time.
The downside is we're falling back to sync reqs. But the real downside is that we would have to have some code inside of the other services that were running
for some time, just to service or kind of handle this new service that is coming online.
So we need to implement an endpoint for those long running services.

2) Direct DB access.
This option is one exception to that rule that we said, which is for every service having it's own private DB. So the instant that the new service came 
online, we could give it direct access to the data store or the DB for other services that it needs and were running for some time.
Downside: 
We're making sync reqs over, which means we're gonna have to implement some code inside of new service to work directly with whatever those DBs of other services
are. Let's say one DB is mysql and other is mongodb. In this case, we need to write some code inside of the new service to interface with a mysql db and a mongodb
db. That's a lot of code.

3) Store events.
The new service could work if it had access to all the events that had been emitted in the past.
In this approach, whenever ANY of our services emits AN event whatsoever over to the event bus, the event bus will send that event out to all the other
services, but it's gonna do sth else at the same time. It's gonna store that event internally(probably not in memory, instead in a DB, because this DB is 
gonna grow to be very, very large over time).
Upside: If a new service comes along or online at some point in future, we can say to event bus: Hey, give me access to all the events you have stored. Just throw them
over and I'll(the new service) decide whether or not I care about them.
So we can use all the SAME code that we've already written to handle these exact events that came from event-bus to sync this new service with other services.
So we don't have to write any extra code.

We're gonna use this option.

So whenever we emit an event from a service, we're gonna store it with our event-bus so that if we ever bring a service online in the future, we can get
access to all those events that occurred in the past.
This also solves the issue with a service going down for some point in time.
So after a service comes back online again, it can take a look at whatever the last event was that it received and it could say to the event-bus:
hey, give me all the events during the time that I was down and event bus could say: Ok, you received (for example) event N, I'll give you everything since then.
So I'll give you event N till the last one.

So now the service that was down is all caught up.

So this option not only does it solve the issue of bringing services online in the future, but it also solves the problem of events possibly
being missed while a service is experiencing some amount of downtime.


# 50-040 Required Node v15+ Update for Query Service:

# 51-041 Implementing Event Sync:
We're gonna take the query service down, then we're gonna create some posts and some comments as well. Then we're gonna launch the query service again and
we're gonna try to make sure that the query service can reach out to the event-bus and tell it give me all the events that have occurred up to this point.

Whenever our query service comes online and it starts listening on port 4002, right after that, it would probably a good time to make a req to event-bus 
and get a listing of all the different events that have been emitted up until this point in time.

# 52-042 Event Syncing in Action:
Stop the query service server. Then create some posts and comments and those events will be stored by event-bus and then as soon as we launch query service, it 
would resync and get all the events that it has missed over that time.


