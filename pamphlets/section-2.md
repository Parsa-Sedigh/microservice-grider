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
