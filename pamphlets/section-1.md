# 01 - Fundamental Ideas Around Microservices

1-001 How to Get Help:
2-002 Course Resources
3-003 Join Our Community_:
4-004 What Is a Microservice_:
A monolith contains routing, middlewares, business logic and db access to implement all features of our app.
A microservice contains routing, middlewares, business logic and db access to implement **ONE** features of our app.

The service A should be 100% standalone and doesn't necessarily require any other service to wrok correctly. So if other services crashes or ...,
service A should work fine.

5-005 Data in Microservices:
One of the challenges when working with microservices is data management between services. 
Data management means the way in which we store data inside of a service and how we communicate that data between different services.

Whenever we are making use of microservices, we're gonna create a separate db for each service(IF the service actually needs a DB).
To access data, we're never going to access data by reaching from one service into another services DB.
So services will never, ever reach into another service's DB.

So under no circumstance the service A will try to reach into the DB for service B.

So:
- Each service gets it's own DB(IF it needs one). This pattern(each service gets it's own DB) is called database per service.
- services will never ever reach into another service's DB 
But why? Why database-per-service?
- We want each service to run independently of other services. Why?
  Because if every service used a common DB, the problems are:
   1- is if anything bad happens to this common DB, ALL of our services are gonna crash.
   2- the scaling of this common db would be challenging. Because we're gonna have to scale up that ONE db instance to give service to ALL the DIFFERENT
      services our app.
  Another reason is that if service A is reaching the DB of service B, if anything goes wrong with the DB of service B, service A is gonna strat to crash.
  Because we have introduced a dependency between service A and service B. So rather than just losing service B, we would now ALSO be losing service A.
  So by giving each service it's own DB and making sure that each service only uses it's own DB, we increase the uptime for our entire system.
- DB schema/structure might change unexpectedly
- Some services might function more efficiently with different types of DB's(sql vs nosql)


6-006 Big Problems with Data:
In microservices, we do not allow a service to reach out to DBs owned by other services.


7-007 Sync Communication Between Services:
If a service relies upon data from other services, it starts to get challenging if we're gonna follow that pattern of DB per service.
Two very general strategies that we can use to solve this problem:
Strategies for communication between services:
sync and async(These words don't mean what they mean in the JS world!).

Sync: Services communicate with each other using direct **requests**. This request doesn't necessarily have to be an HTTP req or a req that exchanges JSON, it is just
some kind of direct request, whatever it's form is, a direct request from service A to service B to fetch some information.
Async: Services communicate with each other using **events**.

Example of sync communication:

Pros and cons of sync communication:

8-008 Event-Based Communication:
There are two different ways of using async communication.
The first way is not great. Because it shares all the downsides of sync communication but it also has additional downsides associated with it.

9-009 A Crazy Way of Storing Data:

10-010 Pros and Cons of Async Communication:
In this course we're gonna focus on second approach of using async communications.
The workflow would be like:
We're gonna have a lot of services and they're gonna have incoming reqs, the service is gonna do sth locally with that req and if it makes a change
to any data, it's gonna emit an event. Other services will then wait on around, listen for events(the events they care) to come in and eventually populate their DBs to
answer very specific queries.


