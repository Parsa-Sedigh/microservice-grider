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

## 108-006 Note on Typescript:

## 109-007 Auth Service Setup: