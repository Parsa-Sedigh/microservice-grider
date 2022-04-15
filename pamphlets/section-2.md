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


