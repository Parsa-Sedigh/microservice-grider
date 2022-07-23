# 09 - Authentication Strategies and Options

## 163-001 Fundamental Authentication Strategies:
Handling authentication, handling the process of making sure that someone is sending this req to us, is logged into our app is challenging in microservices.

Handling user authentication, in other words, giving a user a cookie, a JSON web token or sth similar and allowing them to access other services inside
of our app is a challenging problem that is not really solved. In other words, there is not really a perfect solution to handling this stuff.

So instead, with microservices and in the context of this course, we're gonna try to outline a couple of approaches in handling authentication.

Approaches in how we can say: "Hey user, here's a cookie or here's a token. Give this to us in the future" and how we can decide whether or not a user truly is 
authenticated.

![img.png](../img/section-9/163-001-1.png)

![img.png](../img/section-9/163-001-2.png)

User authentication comes down to how we answer this question: How do we decide whether or not someone is logged-in in a microservices app?

There are 2 approaches for this:
The reason there is the word `fundamental` in pictures, is that there are variations of each of these approaches. All of the different strategies
out there really come down to 2 very funamental principles.

1) **option #1**: The idea is that we allow individual services to rely on some centralized authentication service to decide whether or not a user is logged in.
   The `sync request` in the img is related to world of microservices not the world of JS. In the world of microservices, sync req refers to a direct req
   from one service to another, one that does not make use of events or event busses or anything like that.
   So the downsides of fundamental option #1, really shares all the same downsides as synchronous communication in general. If we used this solution, think about
   what would happen if the auth service just mysteriously went down one day? If that thing crashed or just disappeared, all of a sudden, no one, no service
   inside of our entire app can decide if a person is logged in and that means that any req that requires us to decide if a person is authenticated, is 
   automatically going to fail!
2) **option #1.1(variation of #1)**: This option is very close to option #1 because we're still relying upon the auth service 100% of the time. With this, any req
   coming into our app, would need to go through some central gateway of sorts, that would authenticate the incoming req. For example, 
   whenever user makes a req to purchase a ticket, rather than going directly to the orders service, we would instead have it flow into some
   auth gateway or some auth service of sorts that would inspect that incoming req and decide if the user is authenticated. If they are, we would then
   send the req along to the intended destination, otherwise we would just reject it right away. This still shares a lot of pros and cons of option #1, because 
   we still have a 100% reliance on the authentication service in both cases. If this thing ever goes down, all of a sudden we cannot make a single req that
   requires any kind of authentication. 
3) **option #2**: We're gonna teach each individual service how to decide whether or not a user is authenticated. In this scenario, we have no dependency on 
   outside services, no dep on a gateway or some other service or anything like that. Everything is wrapped up inside of one single service and if a user
   ever makes a req to for example the orders service, we will immediately and instantly know whether or not this user is logged in.
   **Upside** is we do not have any outside dep.
   **Downside** is that we're gonna end up duplicating auth logic between all of our services. But the solution is we can move this auth logic into a shared library that's
   used among all of our services. But there are some much more larger critical downsides of this option. We'll see them in next vid.
   

approach #1:
![img.png](../img/section-9/163-001-3.png)

approach #1.1:
![img.png](../img/section-9/163-001-4.png)

approach #2:
![img.png](../img/section-9/163-001-5.png)

## 164-002 Huge Issues with Authentication Strategies
In authentication solution #1, we had a reliance or dep on some central auth service(#1).

We took a look at a slight variation of that, where we had some central gateway that would block unauthenticated reqs(#1.1).

We also had another one(#2) where we're gonna teach each service how to authenticate an incoming req.

![img.png](../img/section-9/164-002-1.png)

Let's say a user logged in to our app and got a JWT. Now after some time, we ban that user using the auth service which is decoupled from other
services of our microservice. But that user still has a valid jwt and he sends a req to some service in our microservice and that service doesn't 
care about auth service, it's decoupled from auth service. So even though the auth service is 100% certain that this user should not have access to
our entire app, at no point using option #2 do we ever go over to that service(auth) to figure out whether or not this person is authenticated.

That is the core issue with approach #2. Even though we get this fantastic separation of services, we don't have any deps, there are still going to be 
scenarios where we try to update data in one location tied to a user status, but all the other services are not going to hear about that update(like a user
just banned). They don't know. They don't have any logic to be told: Hey, this user is now banned or sth like that, because there's no direct connection between
the two.

![img.png](../img/section-9/164-002-2.png)

So all this authentication stuff is a nasty thing. It is an unsolved problem. There's no single solution out there that is just the right way to do it.
We can try to figure out some clever ways to work around some of these restrictions.

We're gonna use a solution even though it still has some downsides.

## 165-003 So Which Option:
We looked at 2 options for handling the question: whether or not a user is authenticated inside of our app.

Option #1:
**pros**:
Anytime we made changes to our auth state or the access of a user, it would be immediately reflected throughout the **rest** of our services.
So if we said a user was banned, as soon as the next service came and asked about the status of a user or whether or not they were actually signed in, we could
absolutely say this person is banned, do not allow them to access.
But there is a downside too which is in image.

cons: in image

![img.png](../img/section-9/165-003-1.png)

Option #2:
pros: In image
cons:
If some user ever got banned, there is going to be a window or a period of time where we were gonna continue to trust that that user was actually signed in.

In microservices, the async communication really leads to a huge amount of independence between our different services. But we can have a hybird of sorts.
We can have a set up where we've got a ton of async communication going on, but we can also have some little instances of sync communication as well.

For the problem with updating the authentication status of users(like banning them) which can also be a security issue as explained, we will solve it in the future.





## 166-004 Solving Issues with Option #2:
The strategy that we're gonna suggest in this vid, is not gonna be used. Because it's a tremendous of extra work but it's a viable strategy.

1) In this strategy, when we send back that jwt or cookie or ..., we're gonna somehow make sure that that thing is only viable for that next 15 minutes.
How? Well, there are mechanisms around JWT in particular, for making sure that it's really clear that this token is only viable for some set period of time.

Let's now imagine what would happen if user ABC attempted to make a req to purchase some ticket through our orders service.

Flow: They're gonna make a req to purchase a ticket and they're gonna supply a json web token, cookie or ... which let's say it's 30 minutes old.
So in this scenario, we're gonna say that they have some expired jwt. So when they make this req, we're then going to ask if this person is logged in? Because we're
following option #2, we're going to rely upon our orders service to take a look at that jwt and decide if the user is authenticated?
So we're gonna run some logic to look at that token and critically inside this logic, we're going to add sth to say if that jwt is older than 30 minutes, then this person
is **not** authenticated. If they have an expired token, there are **two** ways that we can deal with it easily.

We can either have our orders service(and that logic for auth inside there), attempt to reach out to the auth service and get a new **refresh** token.
So we could reach out directly with a synchronous req to the authentication service, get a refresh token and then when we respond to the overall req(purchase ticket req),
we could **include** the new refresh token and send it back to the user ABC.

That is one possible approach and would allow us to refresh the token all in one req while still acheiving the overall goal.

The nice thing about this approach is that it requires us(if this token is expired), **to reach out to the auth service** and so that would be a time
for the auth service to chime in and say: Hey, this person is banned! Don't allow them access or sth like that.

![img.png](../img/section-9/166-004-1.png)

2) The other strategy we can take here, if we do not want to have the synchronous communication, we could say that if the jwt is older than 30 minutes, 
then we're just going to reject the overall req. So we could return early and send back an error and tell them: Hey, before you make a req to us,
you need to go and refresh your token and so rather than doing the sync req ourselves, we can tell our **client** that they have to go over to the auth service(like
logging in again which will send a req to auth service) and refresh that token on their own and once that's done(once they got the updated token), that would
now be like 10 seconds old, now they can make the follow up req or repeat the same req again to **purchase** a ticket with the brand new refresh token.

So the diagram in case the jwt was expired in this approach would be:
![img.png](../img/section-9/166-004-2.png)

So this definintely solves kind of the issue. Why kind of?
Because we still have a window here, a window of 15 minutes where we could potentially ban a user and then they could **continue** making reqs in the window of those
15 minutes! and so this is where it really starts to get into your personal implementation or the requirements of whay you're building.

You might be building an app where that 15 minutes window is tolerable. You could even take it down to 5 minutes or 1 minute and there might be some window of 
time that you would be happy with and saying: Yeah, if the person gets banned, they can still make reqs in the span of time, I don't care.

But on more secure apps, there cannot be any period of time where a user gets banned and then can make follow up reqs and still be authenticated.
In this case, here's what we would do:
![img.png](../img/section-9/166-004-3.png)

In this approach, we would imagine that an administrator user would make a req to ban user ABC. It would reach auth service and there we would have some
user management logic and we would reach out to our DB and mark user ABC as not having access anymore. So they're banned as far as the auth service is concerned.
**But** we want to reflect this change immediately with all of our services as well and we would probably want to do so using the same communication patterns
as we've seen, in other words, using events as opposed to synchronous reqs. So here's how we would handle this:

Right after we update the DB that that user is banned, we would then also emit a `user_banned` event or sth similar. It'd be some kind of event that says:
Hey, all services out there, anyone who's listening, do not allow this user to access the app. We would send that to event bus and that event would be sent off
to all of our different services. Then inside each of our services, we could take that info out of that event and we could persist it in some very short lived]
cache or some kind of short-lived data store. Something to say: Hey, here's all the users who should be banned and who should revoke access to.

The reason for using short-lived in memory cache, is that remember we don't really want to be storing a list for all eternity of the users who are banned.
Users might get banned or unbanned all the time.

The reason we're saying short-lived is that we can just persist this data for 15 minutes which is the same duration of time as the lifetime of these 
JWTs, because after 15 minutes, we don't need the store anymore. After 15 minutes, our service is gonna immediately know that incoming JWTs are expired, so
immediately they will reject the req. But within those 15 minutes, we can **temporarily** store this info that this user should be banned.

So we only need to persist this list of banned users in each individual service for an amount of time equal to the lifetime of our auth mechanism.

Upside to this approach is that we can immediately ban a user from all of our different services. 

The downside is that we have to have some implementation in each service, for listening to that UserBanned event, storing a list of banned users for a period of time and
then comparing whenever we run some authentication logic.

![img.png](../img/section-9/166-004-4.png)

So this is how we can still implement option #2 and not have any window of security issue whatsoever inside there. But we're not gonna implement this stuff. 
So we can go with option #2 and have a very secure approach.

We will start to implement option #2 in next vid.
 
## 167-005 Reminder on Cookies vs JWT's:
