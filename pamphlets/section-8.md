# 08 - Database Management and Modeling

## 147-001 Creating Databases in Kubernetes:
Whenever a user signs uo to our app, look at 147-001-1

Every service we're going to implement, is gonna have it's own private instance of mongodb and this goes back to the idea of one database per service.
Now install mongoose in the auth service.

Now that mongoose is installed, let's create a new instance of mongodb:
![img.png](../img/section-8/147-001-2.png)

**Note:** Mongodb is just like any other thing that we're trying to run inside of our k8s cluster. We're going to be running mongodb inside of a pod and remember,
we usually do not create pods **DIRECTLY**. Instead, we make use of deployments. The deployment is gonna create a pod for us and in order to communicate with this
pod, we're gonna have to create a cluster ip service to go along with it as well.
In total, we're going to just be writing out one single deployment config file which has the deployment config and the cluster ip service config.

So create a file named auth-mongo-depl.yaml . This is the general naming strategy that we're going to use for our different databases that we create.
So they would be: <name of the service that owns this DB>-<type of DB like mongo>-<we're creating a deployment, so 'depl'>.yaml .

The `template` section in a `Deployment` config, is for pods and it contains:
```yaml
    labels:
      app: auth-mongo
```
is going to be a label that gets applied to the pod and this section:
```yaml
  selector:
      matchLabels:
       app: auth-mongo
```
is how the deployment is gonna find the pods that it actually creates.

When we have: `image: mongo` in Deployment config, we know that on docker hub there are a lot of different images hosted on docker hub for public use and 
there is an image called `mongo` there. So it refers to the mongo image on docker hub.

**Note:** After creating the deployment, we're also gonna create a cluster ip service inside of that same file so that we can actually connect to the pods that are
going to be created. So add in 3 dashes to indicate that we want to put in some configuration for another object.

When we put:
```yaml
selector:
    app: auth-mongo
```
inside of a cluster ip service, that is telling the service which pods it is going to govern access to. So in this case, we're saying find a set of pods or any pod
with the label of: `app: auth-mongo` and that matches with the pods we wrote in the deployment object of mongodb.

The `name` we put for an entry in `ports` section is only for logging purposes.

Why are we using 27017 as port numbers for db?
By default, mongodb listens for incoming traffic on 27017. That's the default port for mongo.
You would normally do a bit of research to figure out what port a given DB or a given application is going to be listening on. To do so, you would look up some
info about the image that you wanna use for that DB or app.

Now to create that DB deployment and the service, we'll do `k apply -f ...` or a better way of handling this would be to just make use of skaffold.
Remember skaffold is gonna try to deploy all the different files we list out inside the `manifests` section.

Sometimes skaffold does not actually see when we create new files with some objects inside them. You can hit ctrl+c then restart it by running skaffold dev again.

Once everything gets applied, we should be able to run: `k get pods` (Remember that you're connected to the right cluster in your context in docker desktop) and then
run k get pods.

So now we're running a copy of mongodb for our auth service.

Now we need to make sure our microservice service has some code inside of it to connect to the mongo instance which is running inside a pod.

## 148-002 Connecting to MongoDB:
Look at 148-002-1.

Right now, we have created a deployment that creates a pod that is gonna run our mongodb instance. We haven't really spoken about the lifecyle of pods in k8s yet,
but suffice it to say: if we ever delete or restart the pod that's running mongodb, we're gonna lose all the data that is loaded into our DB.
Later, we'll fix it and discuss why we're losing this data and figure out some way to give mongodb some persistent storage.

There are many ways that we can connect to mongo instance using mongoose. We're gonna use async await syntax.

If we're running or connecting to a mongo instance that is on our local machine, the connection string would be sth like: mongodb://localhost
But in our case, we're not really connecting to a mongodb instance that is available on localhost, instead, it is available inside of that other pod and 
remember to connect to that other pod, we have to go through that cluster ip service we had created. 

**Note:** Anytime we're trying to connect to a cluster ip service, we're going to write in the name of that cluster ip service where we would normally put a domain and
then put the port of cluster ip service. After port, put the name of the actual DB that we want to connect to. If we MAKE UP A NAME OURSELVES there, mongodb or mongoose will
create the database for us automatically with the name that we chose. Let's name that DB, auth, so mongodb will now create a DB named auth.

As of mongoose v6, do not add options object to that connect() method.

The reason that we defined a separate function named start() is that depending upon the version of node that you're running, you have to wrap all of your
async await code inside of a function which in this case that function is named start.
The latest version of node(in our case the `docker image` of node inside your cluster) can make use of await keyword at the top level(outisde of a function).

## 149-003 Understanding the Signup Flow:
Look at 149-003-1.
![img_1.png](../img/section-8/149-003-1.png)

We want to make sure all the different users of our app have unique email, so check if a user ever signed up with this email address before.

In order to access the mongodb collection, we have to create a mongoose model which is gonna govern access to that collection.

Turns out that TS and mongoose do not really cooperate well. There is a library that gets mongoose and TS work a little bit more nicely.
But we don't use it in this course.

## 150-004 Getting TypeScript and Mongoose to Cooperate:
Here's the issue that we're gonna run into:
![img_2.png](../img/section-8/150-004-1.png)

Mongoose model is a class that is gonna represent the entire collection of users(represents all of our data).

A user document is also a class and is meant to represent one singular user.

The issues are:
![img_2.png](../img/section-8/150-004-2.png)
Ts wants to understand the different args that calling new User() is gonna accept. However, mongoose does not make this easy. Behind the scenes or more
specifically with the mongoose type definition file that we just installed(not required anymore I guess), whenever we try to create an instance of a user,
it is not gonna kinda tell TS about the different properties that we're trying to pass in. So we could make a typo when passing args to new User() (when we're
creating a document) and TS is not gonna complain or help us! Because TS is not gonna be given enough info by mongoose by default.

Issue #2:
At some point in time after we created a new doc, we want to access some of the data within that created new doc, like logging the doc, the issue is
we might end up seeing that that user doc has MORE properties than what we provided to the constructor when creating that doc. So behind the scenes,
mongoose might decide to add in a couple of additional properties to that doc as well, like createdAt or updatedAt or ... .

So the properties we pass into the constructor, don't necessarily match up with all the properties that are created on the user doc. 

This is also an issue with TS, because TS wants to know about all the properties that exist on the document. So we need to tell TS that in addition to properties
we passed to constructor(when we say: new Class(), we're passing args to the constructor of the class), there are going to be some other properties that get created behind the scenes
and assigned to the user as well. So we need to teach TS that there are kinda two different sets of properties. There's one set of properties that will
passed to the constructor and there's gonna be a second set of properties that we should be able to access on the user document that gets created.
![img_2.png](../img/section-8/150-004-3.png)


## 151-005 Creating the User Model:
Create a directory called models and there create user.ts .

A schema is how we're going to tell mongoose specifically, about all the different properties that an entity is gonna have.

The `type: String` in mongoose schema is not tied to TS. This type is 100% specific to mongoose. It does not tell TS **anything whatsoever**.

Now if we play around with the User modal that was created by: `const User = mongoose.model('User', userSchema);`
those two issues will cause some problems.
For example if you say:
```typescript
new User({
    email: '...',
    password: '...'
});
```
still we don't see any error. However, there is no info about the args that are going to be provided to this User constructor being fed into TS right now!
So if we say:
```typescript
new User({
    email: '...',
    pas: '...'
});
```
there's no problem around TS. So TS has no idea about what is going on with the args we're passing to this constructor. So we need to teach TS about different 
properties we're gonna pass into that constructor.


## 152-006 Type Checking User Properties:
Currently, TS is not checking the type of args we're passing to the mongoose models. This is issue #1.

To solve this, create an interface.
Now use that interface to make sure that anytime we try to create a new document out of that model, we're passing in the correct set of arguments or attributes.
To do this, create a new function called buildUser. Anytime we want to create a new user, we're gonna call this function instead of calling new User() . 
So keep this in mind.

So we're not gonna follow the normal way of creating a new doc which is calling new <model class>(). Because we cannot really do effective type checking with TS if we follow
that normal pattern.

So we're putting an extra step to get TS involved in this process of creating a user.

This is how we solve issue #1.

So now anytime we want to create a new user doc or work with users in anyway, we now have to import two different things from that user.ts file. That is not
convenient.

We want to make creating a new user doc, somewhat similar to the normal way of new User() , so we don't have to memorize to call buildUser() anytime we want to create a
new user.

## 153-007 Adding Static Properties to a Model:
Let's write a slightly better way of writing out buildUser function so that's not a separate standalone function. Instead, we're going to get this thing builtin or
included with our User model.

The end goal is be able to say: User.build({...}) . The nice thing about this appraoch is we no longer have to export a separate thing from that user.ts and 
creating a new user is kind of a natural thing. Because we say: User.build() , so build a new user.

To add in a new method to our user model, right above where we create our model from the schema, we say:
`
<schema>.statics.<name of the function which we'll call> = (attrs: <some type>) => {
    return new User(attrs);
};
`

This is how we get a custom function built into a model. We add it to that statics property on our schema.

But this doesn't work! Because now if you say:
```typescript
User.build({
    
});
```
TS will throw: `Property 'build' does not exist on type 'Model<Document, {}>'`.

So this is yet another point at which getting TS and mongoose to work together is just a bit awkward. TS doesn't understand what it means to assign a property
to the `statics` object of a schema. We have to give more info to TS.

To fix this, create an interface that describes the properties that a User model has. So we're gonna write an interface that's going to essentially tell TS that there's
going to be a build function available on that User model and we're also gonna tell it what args are required to call it as well.
Create the `UserModel` interface.

Now because our interface is extending another one, the properties that are defined in the interface we're extending, are gonna available BY DEFAULT(this is the effect of 
extend keyword) on our new interface, but we can also define same properties but with different types for our new interface.
So when we have:
```typescript
interface UserModel extends mongoose.Model {

}
```
So now UserModel has all of the properties and their types that are available on mongoose.Model interface.

Now to tell TS about the existence of UserModel interface, add the USerModel as the SECOND type parameter of `mongoose.model()` so now we have: `mongoose.model<UserModel>()`.

We have solved issue #1. We have not really solved anything related to issue #2 where we needed to somehow tell TS and mongoose that the set of properties
that we pass into creating a new user, are different than the properties that END UP on a user doc.

The UserModel interface represents what the entire collection of users look like(so it has mongoose.Model properties + the ones we define(in this case, `build` method))

## 154-008 Defining Extra Document Properties:
Create another interface named `UserDoc` which represents what properties a single user DOCUMENT has.

If we ended up adding in additional properties to the user document, or if mongoose automatically added those properties in(like createdAt and 
updatedAt), that `UserDoc` interface is where we list them. **But** in our case, we're not telling mongoose to add in those properties for us right now.
So don't add createdAt and updatedAt. We have not createdAt or updatedAt as properties of a user document, so don't add them. Even if we know mongoose
is gonna add in those properties automatically for us. Now to tell TS that: Hey, it's OK if we want to access the proerties it's gonna automatically add to a document for us,
all we have to do is to now add them to UserDoc(now that you learned, you can add them).
But for now, because we really don't USE those automatically added properties, let's remove createdAt and updatedAt from UserDoc interface, again, because 
we don't USE them in OUR CODE(but in runtime, they are in the user document, we just didn't define them at compile time in the interface).

Now add this new interface to UserModel's build method and first type param of mongoose.model() .

Now we can access user's email and password. So we can access document's properties. If we didn't use UserDoc interface, we would not be able to access these properties safely.

## 155-009 What's That Angle Bracket For:
When we have sth like:
```typescript
const User = mongoose.model<UserDoc, UserModel>('User', userSchema);
```
you can think of those types in angle brackets as being some arguments to the model() method, but instead of being an actual value, it's a type.
So you can think of these as being types being provided to a function or method as arguments.

Generics allow us to customize the types being used inside of a function, method, a class or an interface.

## 156-010 User Creation:
Currently, when we send a response back when user was created(signup route), the password was included in the response which we do not want to do.

## 157-011 Proper Error Handling:

## 158-012 Note on Password Hashing:

## 159-013 Reminder on Password Hashing:

## 160-014 Adding Password Hashing:

## 161-015 Comparing Hashed Password:

## 162-016 Mongoose Pre-Save Hooks: