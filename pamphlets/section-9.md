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
Each service should understand what authentication is and how to determine whether or not someone is logged in?

Now we're going to establish exactly how we're going to prove that a user is authenticated.
In other words, are we using JWT or cookies or what?

### difference between JWT and cookies
Cookies:

Imagine a browser making a req to a server. When the server sends a response back to the browser, it can optionally include a header of `Set-Cookie`
and then for that `Set-Cookie` header, it can provide some kind of value. This value can be a string that contains any info that we want.
That little piece of info is then going to be automatically stored inside the browser. Then, whenever this browser makes a follow up req to the same
domain with the same port, the browser is going to make sure that it takes that little piece of info and appends it onto the req as a cookie header(that
little piece of info in diagram is 'ajskdp3'). It will be automatically sent to the server.

The idea of saving a cookie is that some arbitrary piece of info, doesn't really matter what this info is, it can be anything.
That piece of info will be automatically stored by the browser and automatically sent to the server at anytime we make a followup req.

![img.png](../img/section-9/167-005-1.png)

### JWT:
With a JWT, we're going to take some arbitrary piece of info that we refer to as the `payload`. It can be some object that has maybe a userId or ... any info.
We're then going to take that payload and throw it into a json web token creation algorithm. It's then going to spit out our JWT that looks like an encoded
string like in diagram.
Stored in that encoded string(JWT), is that original payload. We can easily take that encoded string, throw it into some kind of decoding algo and extract
the original object(payload). So at any point in time, we can always access the info(payload) that is stored inside the token.
![img.png](../img/section-9/167-005-2.png)

Once we have this token, we eventually do need to communicate it between the browser and server. There are a couple of different methods that we can use to do
this communication:
Whenever the browser makes a req to the server, it's going to want to include that JWT in one way or another so that the browser can prove that it is
authenticated or it's logged in with this particular server.

To communicate that token over to the server, some very common approaches are:
- to include an authorization header that has the JWT inside of it(like: `Bearer <JWT>`)
- we can just throw the entire token inside the body of the req(assuming that it's a POST req or a PUT req or a DELETE and ...)
- or alternatively, we can also kinda mix and match here and take that JWT token and store it inside of a cookie as well. So the JWT will be managed
  **automatically** by the browser included on all followup reqs.

![img.png](../img/section-9/167-005-3.png)

Differences between cookies and JWTs:
![img.png](../img/section-9/167-005-4.png)

- Cookies are a transport mechanism. They're a way of communicating info between the server and the browser and they do not necessarily do anything closely
  coupled to authorization. Yeah, we use them for authorization, but that's not necessarily the primary purpose(goal) of them. It's not the only thing
  they can do.
- We can use cookies to move any kind of data between the browser and the server, so it doesn't have to be authentication related stuff, it can be tracking information,
  it can be some kind of visit counter, just about any kind of data we can easily store inside that cookie.
- cookies are automatically managed by the browser. We as developers, specifically on the browser side of things, don't really have to worry about managing them
  in any way. All we have to do on the server is set a cookie and then we can pretty much guaraneteed that will always come back in all follow up reqs.

JWT:
- these are all about authentication and authorization. That is what they are intended to serve.
- inside of a JWT< we can store any structure of data that we want. It's traditionally going to be an object with some key value pairs.
- JWTs have to be managed manually by developers on the frontend, unless we're storing that JWT inside of a cookie.

So the cookie is just a transport mechanism. It's sth that's going to hold info. It can be any kind of info, it's not necessarily tied to authentication.
JWTs on the other hand, traditionally always used for authentication and authorization.

So we need to decide based on pros and cons, which one is more appropriate or combination of the two is most appropriate for handling auth inside of a microservices 
architecture.

## 168-006 Microservices Auth Requirements
Cookies and JWTs are two very different things. The implication here, usually when we say that we're using cookie based authentication, the implication is that
we have a cookie that is going to store some kind of encoded token inside of it and that's gonna be encrypted in some way that the user cannot access the
information inside there.

Now let's see our app requirements and see which of these two approaches is gonna satisfy those requirements.

![img.png](../img/section-9/168-006-2.png)

Inside of our ticket purchase logic, we probably needed to figure out whether or not that user is logged in and OK, we can use a JWT or some cookie based
approach for this(look at green box) and then after that, we probably needed to decide whether or not this user can purchase a ticket. So maybe whether or not
they have billing set up in their account, whether or not they have a credit card or ... . This implication that we need to somehow figure out whether or not
this user or details about this user, really implies that our authentication mechanism needs to tell us information about the person who was just authenticated.
In other words, we can't just have some mechanism that says: Yes, this user is authenticated or no, they're not. 
Instead, this authentication mechanism has to tell us: Yes, this person is authenticated **and here is some information about them: here's their
userId, there's their email, here's whether or not they have a credit card tied to their account.**
So we need to have the ability to store info inside of our auth mechanism. This is requirement #1.

Requirement #2:
Imagine a new scenario:
![img.png](../img/section-9/168-006-2.png)
Imagine that our app has the ability to create free coupons. So these are coupon codes that allow people to get free tickets.
Of course we probably do not want any arbitrary user to be able to access this route handler and start creating free tickets.
So maybe we want to limit this route to admin users. So only admins can make a req that will create a free coupon code.

In that route handler, we need to check whether or not this person is authorized to create free coupons. So we need to decide wherer or not this is an
admin user.

This step is really implying that our auth mechanism has to include some amount of authorization information. We need to know more info about this user. Are
they user? Are they admin? What's their role?

![img.png](../img/section-9/168-006-4.png)
This diagram shows one way to solve the updating status of a use(like being banned) is to have an auth mechanism that would expire after some set of time.
So this is implying that whatever mechanism we're going to use, it has to have a built-in way and a super secure way. In other words we would not want a 
user to be able to tamper with this in some way of expiring our auth mechanism after a set period of time, a very precise period of time.

![img.png](../img/section-9/168-006-5.png)
One of the big benefits of using a micro services approach, is that each of our different services can be built using a different backend language.
This is implying that whatever auth mechanism we decide to use, it has to be easily understood by many different languages.

Also, given the fact that each of those different services that might be written in different languages, are going to need to somehow authenticate
a user, we probably do not want to force any storage requirements on these services as well. In other words, if we're deciding on some auth mechanism that is going
to require us to have some backend DB just to store info to support that auth mechanism, that would probably be bad. We don't want to force for example
our orders service to have some kind of specialized DB to store information just to support our auth mechanism. That is not good.

Summary of the requirements for our auth mechanism:
![img.png](../img/section-9/168-006-6.png)

So:
- first we have to store information about a user like userId and ... , within the auth mechanism itself(JWT or cookie) which is closely couped with the last item.
  So if we're storing info about that user, it has to be inside the auth mechanism and it must not require us to have some kind of backing data store.
- with second item, we can say whether or not this person is an admin, a normal user or ... .
- ...

![img.png](../img/section-9/168-006-7.png)

All these requirements are steering us towards a JWT approach. Why?
- because JWTs can store any arbitrary information we want about a user.
- There are even built-in properties of JWTs that lend it to a handling authorization very well.
- JWTs can encode expiration info. you might be saying: Hey, cookies can do that as well, we can have cookies expire after a set period of time.
  Well, not quite. We can set some expiration on a cookie but that is us kind of politely asking a browser to expire the cookie. Cookie expiration is handled
  by the browser. So the browser is going to receive a cookie. It's going to see that we ask the browser to expire that after, say, 20 days and after
  20 days, the browser will expire that cookie for us. **BUT!!!** a user could very easily copy that cookie info and just ignore the expiration date on there and continue
  using that cookie in the auth mechanism or auth info inside there at some point in time in future, even beyond that expiration date. 
  A JWT however, is going to encode the expiration time in itself and as soon as the JWT expires, then that's it. It will not be valid anymore.
- JWTs have fantastic support between different languages. If we use solely cookie based authentication where we kind of create some kind of custom token and
  store it inside of a cookie, it turns our that a lot of these different custom cookie implementations will vary significantly between different languages.
  It would be kind of hard to create a cookie that is signed securely(so has some actual kind of secure or encryption around it) using nodejs and then reuse that
  over in the ruby on rails world.
- JWTs do not strictly require us to have some kind of backing data store to identify this user.
  Now as opposed to cookies, you will see a lot of people who will use cookies, they're going to try to store, just say a session id inside the cookie that
  refers to some session that is stored on a backing server. That is not strictly required. We can store again any arbitrary info we want inside of a cookie, but
  you're gonna see that a lot of people say that in cookie based authentication, you're going to store some kind of session id and so of course that implies that
  you will have a backing data store on the server. Which again, we probably do not want to have.

So we're gonna use JWTs.

We're still not done here. Remember, when we decide to use a JWT, that's not the end of the story, because JWTs are just an auth **mechanism**. They're a 
piece of data that proves that someone is logged in and store some info about the user. We still have to somehow communicate this info between our
browser and our server. We can do it either manually to a degree using that top yellow box(headers), or sticking it(the JWT) into the body of the req(middle yellow box),
or we can have browser handle it for us using still this kind of cookie based idea(that bottom yellow box)!
![img.png](../img/section-9/167-005-3.png)

In the bottom box, we're saying that we're going to use a JWT token to **store**  our auth **info**, **but** the cookie is going to be the **transport** mechanism.
THat;s how we actually move the cookie around between the browser and server(that cookie in itself has a jwt)

So we still have to decide how we're going to move this token around(using one of those 3 options)?


## 169-007 Issues with JWT's and Server Side Rendering:

## 170-008 Cookies and Encryption

## 171-009 Adding Session Support

## 172-010 Generating a JWT
