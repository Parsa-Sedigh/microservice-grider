# 10 - Testing Isolated Microservices

## 188-001 Scope of Testing
We have done testing with postman, but no automated test.

How we handle testing in a microservices type environment?

Testing of microservices is a big question of what the scope of our tests are gonna be?
We need to decide what we're trying to test?
The list in the image is not an exhaustive list.

The first blue box in top is unit test.
![img.png](../img/section-9/188-001-1.png)
![img.png](../img/section-9/188-001-2.png)

What does our app look like right now? and some parts to represent what it might look like in the future?(orders, payment and ticketing are not yet implemented).
![img.png](../img/section-9/188-001-3.png)

Unit test is like testing that red box:
![img.png](../img/section-9/188-001-4.png)

The next one is for example a req coming in and had it flow through the requireAuth middleware and then up to signup route handler. It means a test that spans across
differet pieces of code.
![img.png](../img/section-9/188-001-5.png)

How multiple components(programs of our app) work together. For example how our auth service interacts with mongodb.
![img.png](../img/section-9/188-001-6.png)

The largest thing we can really test in one go, would be to test out different services work together:
![img.png](../img/section-9/188-001-7.png)

If we want to test how the orders service and the ticketing service(multiple services) interact with each other, we need to think about how to construct some kind of
env quickly and cost effectively to actually test these stuff. How would we do that? Would we launch some kind of test k8s cluster and then launch the orders service and
ticketing service inside there? How would we send reqs to these? How would we assert the results of any req we make? It's a complicated thing when multiple services work
together.

Therefore, we're not gonna focus too much on how we can test different services working together, instead, we're gonna try test these different services more or less in isolation.
Meaning we're gonna test each service individually.


## 189-002 Testing Goals
**These are not types of test but rather testing scenarios for our app.**

The kinds of tests we're gonna write as we're looking at 1 service:

The first lind of test that we're gonna write out, is gonna to try to test basic request handling. For example, we might want to try to make sure that if we make a req
to our auth service to signup for sth, then perhaps we should get back a res with a cookie that has a JWT inside of it, or we might want to try to assert that we write
some data into our mongodb DB. So the goal is that we can issue a req to a service and then make sure that it gets handled in some very particular fashion.
![img.png](../img/section-9/189-002-1.png)

The second goal for our test we're gonna write, is gonna be more of a unit test style approach. For example, in this test, we might take a look at a very particular model inside
of app and try to test some functionality around it(make sure they function in a particular way).

![img.png](../img/section-9/189-002-2.png)

The final scenario is to test the receiving and emitting of events inside of our service. Currently, our auth service is not sending out or receiving any events and we
haven't even discussed event handling inside k8s env yet. So we're not gonna worry about test goals #3 for the auth service, but for our
later services we put together, we will be writing out some code to make sure that we can receive incoming events and to make sure they get processed in some particular way and
make sure that we can issue(send) events to the outside world(our microservice world) as well.
![img.png](../img/section-9/189-002-3.png)

Test goal #3(emitting and receiving events) is how we're really gonna accomplish kinda simulating different services working together. So even though we're not going to directly
launch, say an orders and ticketing service at the same time and try to test them together, this idea of testing events or receiving them and emitting them, is very
similar in nature and it will achieve a similar goal.

![img.png](../img/section-9/189-002-4.png)

How are we gonna execute these tests? In other words, when I say: let's run our tests, how do we do that?

We're gonna take a simple and direct approach.

We're gonna run a command like npm run test and that's gonna start up our server on our local machine or our individual service and then try to run some number of tests against it.
This is implying that our local env is capable 100% of running each of these services and that probably makes a lot of sense right now. Think about auth service: What deps or what software
do we need to have installed on our local system to run the auth service?

For now, we just need nodejs and a copy of mongodb, that's all we have ot have installed on our local system to run the auth service. **But in the future**, you might be working on way more
complex projects. Projects where your services have way more deps. For example you might be trying to test a service that requires you to have a particular OS or a particular and complex
and hard to run and hard to setup DB. So even though our current approach is gonna be just fine for this current app, in the future, it might not work out well.
So the tutor is gonna show us some way that we can still run tests easily, but it's more complex in nature, but it can also handle way more complex setup requirements.
So in future, we're gonna learn a guide on how to handle more complex testing in the future.
![img.png](../img/section-9/189-002-5.png)

## 190-003 Testing Architecture
For auth service, we're gonna focus on that testing goal #1. So we're gonna write some tests that are gonna try to send a req off to our service and then make some
assertions about the response or make sure that some data was written into DB.
![img.png](../img/section-9/190-003-1.png)
Note: The boxes on the right side are things that we tell to jest to do.

Add a new script to our package.json named test, whenever we run that command, we're gonna startup a test runner called jest(to **execute** tests inside of our project).

When jest starts running, we're gonna tell jest to do the following things:
- start an in-memory copy of mongodb. So this is gonna be a copy of mongodb that's running in memory. In other words, you do not have to install mongo directly onto your machine.
Now even if you already have mongo installed on your local machine, we're still gonna use this in-memory copy. There's a couple of reasons for that.
- start our express app.
- use supertest to make fake reqs to our express ap
- run some assertions to make sure that the req did exactly what we expect it to do.

In order to get supertest working well, we're gonna refactor our project a little.

![img.png](../img/section-9/190-003-2.png)

In supertest, we have to have access to express app variable which is currently defined in our index.js file.

We have to require the express app from that `index.js` file over to some testing file so that we can access to that app variable. There's a problem with that.
In `index.js` we have code to tell express app to start listening on some given port(like 3000). Why is this relevant?

We might want to run multiple different tests on the same machine. We might run tests against auth service, at the same time against orders service. The auth service and orders
service might both be trying to listen on port 3000. So if we try to require in our express app from that index.js file and both the auth service and ticketing service
or ... are listening on the same port, we're gonna end up with an error that we're not able to run tests from different services on the same machine at the same time just because
they're both listening on the same port. To get around this, we're gonna rely upon default behavior inside supertest. In supertest, if server is not listening 
for connections, then it will automatically start to listen on an ephemeral port. Ephemeral port here means that supertest is gonna pick a random available port
on your computer and have the express app start to listen on that port and this feature will allow us to run tests from multiple different services at the same time and not have to
worry about this port issue. However, if we want to use that functionality, that means that inside index.js , we can't have any code that's gonna have the express app
automatically start to listen on port. So what do we do?

![img.png](../img/section-9/190-003-3.png)
![img.png](../img/section-9/190-003-4.png)

We need to split index.js into 2 different files. We're gonna create app.js and there we create express app and export it. The express app that we create inside app.js is not gonna
be listening on any port. Instead, we just export the app and it can then be safely be used by supertest. We will also import it in a new index.js file and inside index.ts is where
we're gonna have some actual code to start up our app and stat listening on port 3000.

So whenever we want to **actually** run our app inside of a dev or production env, in other words, whenever we want to really start up our server for real, we're still gonna
execute index.js and it will pull in the app variable that is defined in app.js .

So we're gonna have a separate file that exports just the app variable itself, so we can still use it safely inside that test file.

Besides supertest, there are some huge benefits from creating our express app inside it's own file(like app.js) and then doing some actual startup logic inside of index.js file.

## 191-004 Index to App Refactor
app.js is responsible for creating the express app, wiring together all the middlewares, route handlers and ... . Then in index.js we're gonna actually start up the app using
the app exported from app.js along with all the other associated things that need a connection such as mongoose and ... . 

So app.ts does not start up the express app, it just configures it(the app variable).

![img.png](../img/section-9/191-004-1.png)
![img.png](../img/section-9/191-004-2.png)


## 192-005 A Few Dependencies
In auth service, run: 
```shell
npm install --save-dev @types/jest @types/supertest jest ts-jest supertest mongodb-memory-server
```
You'll see as we install **mongodb-memory-server** package, there'll be a rather large download to go along with it(a file that is 80MB large).

The reason we're running a copy of monogdb in memory, is so that we can easily test multiple DBs at the same time. Remember, **we want to be able to run tests for different
services concurrently** on the same machine. That might be eventually start to be challenging if we're having them all connect to the same instance of mongodb.

So rather than having all these different services connecting to the same test instance of mongo, we're going to instead create a mongodb memory server or an instance of mongo in
memory for each of these different services we're testing , that's gonna end up running much more quickly.

Now that 80MB download is sth that we don't want to repeat every single time that we try to build up our docker image or the service like auth. The only thing that's going to be
running inside of our docker container or the service like auth is the actual express application. **We have said that we're not going to be running our tests inside of that image
at any point in time.** So we don't need to install any of those deps like jest or mongodb-memory-server or supertest or ... , as we're building that image. That's why
we installed them as development deps. We're now going to our Dockerfile and we're gonna update it to make sure that whenever we build an image for auth service,
we do not attempt to install these dev deps and again, it's because that we can avoid downloading that 80MB file or other deps that we don't want and add them as dev deps every single
time we have to rebuild our image.
 
For this, use npm install with `--only=prod` option.

So now if we ever decide to install mode deps into our project(which causes the image to be rebuilt) or change any other file that's going to cause the image to be rebuilt, we won't have to
sit around and wait for that 80MB download.

## 193-006 Test Environment Setup
Add a script called `test`.

The `--no-cache` flag is related to our attempt to try yo use TS with jest. Jest doesn't have TS support out of the box and sometimes jest is gonna get confused, understanding
when we change a TS file. This no-cache flag is gonna add in just a little bit of a fix that's gonnna catch some issues that you're going to see they arise any time we try
to change a TS file and jest sometimes doesn't recognize those changes. So the --no-cache flag is to address this issue.

Add `jest` config in package.json . 

About `"preset": "ts-jest"`: We know jest doesn't understand what TS is, for this, we installed a dep called ts-jest that is going to add in TS support in jest for us.

With `setupFilesAfterEnv`, we're going to tell jest to run a setup file inside of our project after it initially starts everything up. Inside that setup file we're gonna
put together a couple of different helpers just to make facilitating writing our test a lot easier and that file is called `setup.ts` .

The imported app in that file is the express app.

In that file which would run before all of our different tests start up, we're gonna create a new instance of that mongo-db-memory-server which is gonna start up a copy
of mongodb in memory which is gonna allow us to run multiple different test suits at the same time across different projects without them all trying to reach out to the same copy of mongo.
Mongo-memory-server also gives us direct memory access or direct access to the mongodb DB which is gonna be a bit handy for a couple of other reasons as well. 

`beforeAll` is a hook function. Whatever we execute inside it, it's gonna run before all of our tests start to be executed. There, we're gonna start up the mongodb-memory-server and
then connect mongoose to it.

The options that we pass to `mongoose.connect()` are just to make sure mongoose doesn't complain about some related stuff or behind the scenes stuff going on.

`beforeEach` is gonna run before **each** of our tests. Before each test starts, we're gonna reach into mongodb DB(in memory mongo DB actually) and we're gonna delete
or reset all the data inside there. To do this, we're goonna use mongoose to take a look at all the different collections that exist inside mongo and delete all those
collections.

Also after we finish running all of our different tests, we need to stop that mongodb-memory-server and also we tell mongoose to disconnect from it as well. We do this in a `afterAll`

## 194-007 Our First Test
To write a test for signup route handler, create a folder in routes folder named __test__.

When we use jest, we follow the convention that if we're trying to test a given file, we make a folder inside the same directory of the file we're trying to test it and
that new folder that we create should be called __test__ and inside that __test__ directory, create a file that has the exact name of the file that we're
trying to test, then `.test.ts` .

If ou try to run the signup route handler test, the test is gonna fail because there we have process.env.JWT_KEY and previously we checked for existence of this env variable
in index.ts file but in our tests, we have splitted our index.ts and app.ts and we're no longer making sure that this env variable exists and it doesn't exist actually because
previously it was defined inside of a pod(we defined it inside of a k8s deployment config file) and in our test env we don't have that env variable.

There are many different ways to define a env variable in a kind of test environment. The way we're gonna do it, is not necessarily the best way of doing it whcih is
in `test.ts` and in `beforeAll`, we're gonan directly set the env variable!

Now the tests are passing.

## 195-008 An Important Note
**Note:** Jest by itself doesn't have support for TS out of the box. So we're using that extra little library called ts-jest which gives jest the ability to understand TS.
Now this ts-jest library or sth inside jest or sth along this entire toolchain , sometimes does not work correctly. So sometimes you're gonna see a test failing, then you fix it and
then you see the test is still failing. It's hard to replicate it.

To fix this, restart jest by hitting ctrl + c and rerun the test script and jest should see the updated file. So again if you start to make changes and
you don't see them reflected in the test suite, just restart the related test script.

## 196-009 Testing Invalid Input
The very first time you startup the test suite, it's gonna take about 8-20 seconds. That's because that in-memory copy of mongodb just takes a bit to load.

You can hit **enter** in terminal to re-run your tests.

By using request().... of supertest, you just need ot `return request()` or `await request()`, either one is fine.

## 197-010 Requiring Unique Emails

## 198-011 Changing Node Env During Tests
The session object(req.session) is gonna turned into string by cookie-session. The cookie-session middleware is then gonna attempt to send that cookie back to the user's browser inside
the response. Precisely, in order to send that info back, the cookie-session middleware is gonna set a header on the response. This header has the name of `Set-Cookie`.

So to make sure that we actually generate a JWT and send it back inside response, let's make sure after singing up successfully, our res has a header of `Set-Cookie`.

but the test will fail. Why?

When we wired up the `cookie-session` library in app.ts , we used `secure: true` which means that cookies are only gonna be shared when someone is making a req to our server over an HTTPS connection.
When we use supertest, we're not making an HTTPS connection, instead, we're making plain HTTP reqs. So we're not making secure reqs in the test env.

So we could either figure out how to make a secure req, or alternatively, what would be way easier to do and way more time effective, we can decide to change that `secure: true` to false if
we're in a test env. We're gonna use `NODE_ENV` env variable. Whenever jest runs our tests at the terminal, it's gonna set that env variable to `'test'`



## 199-012 Tests Around Sign In Functionality

## 200-013 Testing Sign Out

## 201-014 Issues with Cookies During Testing
In current-user.test.ts , after signing up, we expect that we're authenticated in app, but that's not gonna happen(because we get a currentUser of null when hitting
the /currentuser endpoint). Why?

We've been working in the browser and postman till now. The browser and postman have some functionality included inside them to automatically manage cookies
and send cookie data along with any followup reqs to our server. However' we're using supertest inside of our testing setup and supertest by default is not gonna
manage cookies for us automatically. So while we do get back a cookie when we signed up successfully, that cookie does not get included with the followup req.

So we need to find a way to send authenticated req to our services.

## 202-015 Easy Auth Solution

## 203-016 globalThis has no index signature TS Error
In the upcoming lecture (and later with the ticketing, orders and payments services) you may end up seeing a TS error like this in your test/setup.ts file:

Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.ts(7017)

To fix, find the following lines of code in src/test/setup.ts:

`declare global {
    namespace NodeJS {
        export interface Global {
            signin(): Promise<string[]>;
        }
    }
}`
change to:
`declare global { 
    var signin: () => Promise<string[]>;
}`
## 204-017 Auth Helper Function
Create a function assigned to the global scope in setup.ts , so that we can use it in all of our test files. You don't have to use a global function for this,
we're just doing it for convenience, so we don't have to add in repetitive import statements into all of our test files. If you don't want to attach it to global scope,
you can create a auth-helper.ts or ... inside the `test` directory.

This new global function is not gonna be available inside of our normal app code. Because we're gonna define it in setup.ts file , it will only be available inside of our
app in the test environment.

We could return the cookie in this new function and anytime that we need to make an authenticated req, we say:
```ts
const cookie = await signup();
```
and then we can make the req that needs cookie.

We need to tell TS that there is a global `signin` property. So use a `declare` statement in that setup.ts file using `NodeJS` namespace.

You could name that `signin` function, `getCookie`.

So we're attaching a function to global just to avoid an additional import statement at the top of the files, if you don't like this, create that signin function in a separate file
and then import it into the test files.

## 205-018 Testing Non-Authed Requests

