# 07 - Response Normalization Strategies

## 128-001 Creating Route Handlers:
Look at 128-001-1. Each route handler gets it's own separate file.

In the route files, we're gonna set up a router, which is an object that we can use to associate routes with and then at the very bottom,
we're gonna export that router and rename it at the same time. The reason we are renaming that export with the `as` keyword, is that we're going to
end up with many different routers inside of our app. So they can't all be called router.

We COULD just export a function that receives the req and res and next and error and then associate those functions with our app back in the index.ts file.

## 129-002 Scaffolding Routes:

## 130-003 Adding Validation:
When we have const {email, password} = req.body; in a route handler, we're ASSUMING 100% that there's going to be an email and password on the body.
So we're assuming that email and password are actually DEFINED and they actually CONTAIN an email and password. It's possible that a user send us some
malicious req or even just a faulty request(one that doesn't include an email or password in the body of req). 

**Note:** So we don't want any code that's just going to **assume** that a user is providing the right stuff as expected.

We're gonna use **express-validator**.

Validation is the process of making sure that information has the correct structure and form, whereas sanitization is going to **change** that information is some
way. So it might try to remove certain types of characters or ... .

When we install a new package, the package.json is gonna change. Skaffold is watching all the files inside of our project directory and as soon as it 
sees a change to a **non-synced** file(like package.json file), that's going to decide to automatically rebuild our image and then update the associated deployment and
then we can see the updated pod.

With express-validator, in addition to validation and sanitization on body, we can do those on url params and url search params.

In express-validator, trim() is a sanitization step that's gonna make sure that there are no lodaing or trailing spaces on that field.

The withMessage() in the password field, is gonna popup in case the password is not present or doesn't have that supplied length.

As soon as we added in that array for those express-validator middleware, we got some errors on req and res. Add some type annotations to those
two.

Now we need to actually consuming that info coming out of validation. In other words, we never seem to take any actual errors that might coming out of that validation
step and sending them back to the users.

## 131-004 Handling Validation Errors
Currently, we're doing some validation, but we're not actually communicating the results of that validation back to the user.
In the validation step(where we have an array of express validator functions), if there's anything wrong with the fields, those functions(like body()) are
going to append some properties onto that req object. So we use the validationResult function to pull that validation information off.

For testing signup route, make sure you have a Content-Type: application/json in the headers and in the body tab, make sure it's in the JSON format.

## 132-005 Postman HTTPS Issues:
Hi!

Got an error when testing the route handler in the last video? If so, do the following:

Open the Postman preferences. On MacOS, the hot key for this is CMD + ,

Find the setting called SSL certificate verification

Change this setting to off, as shown in the screenshot below

Look at 132-005-1.

## 133-006 Surprising Complexity Around Errors:
Look at 133-006-1.

Currently, we're not following a standard. We're using express-validator and it doesn't follow a standard.
Look at 133-006-2.

Now the reason that that is relevant, is that we're in a microservices architecture and remember at some point we would have many other services.
The other thing that's really relevant is that these other services could be built using completely different languages and frameworks.

Chances are that if we ever have our react app submit some invalid data to a ruby on rails app, the ruby on rails app is probably gonna give back some 
response that looks very different than the one that we just got back from express-validator response.

In a microservices environment where we're going to have multiple different languages, multiple different frameworks running on all those different services.
Whenever we submit some invalid data to these different services, we're gonna get back a error response that might have a very different looking STRUCTURE.

This is a huge problem. Because we're assuming that all these very different response strcutures are going to be going back to the same exact react app.
So inside of our react app, we're gonna have to encode the knowledge of how to work with these different kinds of error responses.
The react app is going have to know that if it is making a req to the auth service, it should expect a error response that looks like the one in digaram.
The react app will also have to know what if it is making a req to the orders service, it should expect a response that looks like that one.

So the engineers of front end app are going to have to have a very in-depth knowledge of exactly what kind of error structure they're going to get back from
each of those very particular services and that's not a responsibility that we want to throw on engineers that are building the react app.

**Note:** We need to be sure that every single service is going to send back a identical looking structure, anytime there is some error that occurs.

## 134-007 Other Sources of Errors:
We need to make sure that we've got a consistent structure for every kind of error response we ever expect to send back from any service.

In 134-007-1, we're saying we have no idea WHERE sth is going to go wrong inside our app. We might have sth go wrong from express-validator, it might go wrong
during some internal validation and it might go wrong even at the database access level and in all these different scenarios, we need to somehow capture
these errors as they are occurring and send back that identical looking structure. 



## 135-008 Solution for Error Handling:
Look at 135-008-1 and 135-008-2.

If we have a middleware function with **4 args**, express is gonna assume that this is a function meant to handle errors that can occur.

## 136-009 Building an Error Handling Middleware:
Create a folder called middlewares and then create a file named error-handler.ts .

The #1 goal inside that error handler is to make sure that we always send back a very consistently structured message. Because again, we do not want our
react app to have to be trying to figure out exactly how to handle all these 30 or ... different kinds of errors.

We have multiple places where our code can fail, multiple places where we can throw errors and no matter what, we're always capturing those errors, thanks to
the fact that express is gonna automatically capture any error that we throw inside of a route handler and throw it off to our error handling middleware.

## 137-010 Communicating More Info to the Error Handler:
**Note:** Whenever we create a new Error and throw it, whatever string we provide as the first arg to that Error constructor, is assigned to the `message` property
of the error.

Look at 137-010-1.

We can communicate a lot of info from our req handler(controller) to our error handling middleware, by attaching that info to the Error object in some way.
Right now, we're really just limited to that message property. In other words, we're limited to a simple string. Ideally, we would send back a lot more info
to the user than just that simple string.

Look at 137-010-2.

If we were in JS land, we could say:
```js
const error = new Error('...');
error.reasons = errors; // create a new property on error object and attach the result of validationResult() which we named it errors in this case.
throw error;
```
then we can access that reasons property inside the error handler and use it there to send back a good structured error response.

So this is ideal, this would give us the ability to share a lot more info about what just went wrong with the user, compared to what we have right now,
which is only to send a string to the error handler.

But in TS. we can't just assign magic properties made up on the fly(like reasons) to any old object that we want to assign stuff to.
An Error type, in ts and js doesn't have a reasons property. So ts is gonna throw error.

We need to somehow communicate a lot more info from the req handler to the error handler, we have to do it using the Error object, but we cannot really use 
this simple approach which is creating on the fly properties.

## 138-011 Encoding More Information In an Error:
Here's what we want to do:(Look at 138-011-1).

Anytime you say you want sth like X, but you want to add in some more custom properties, that is usually a sign that you want to subclass sth.

We're gonna take the built-in Error object and subclass it into two separate sub-classes. Look at 138-011-2.

The reason that we're making both these subclasses, is that when we create the subclass, we can add in some additional custom properties to it that will
further clarify the reason for the error. Again, we're trying to use these custom subclasses to communicate much more info from the req handler to that
error handling middleware.

Look at 138-011-3. First we start off that the top(request handler section). 

## 139-012 Subclassing for Custom Errors:
Inside src, create a new folder called `errors` and there, we're gonna write the different custom subclasses of Error.

In the constructor of RequestValidationError, we're using the keyword `private`, because we want to take that property(actually argument) and assign it as a property to the 
overall class. So writing out the private keyword in the () of constructor, is equivalent of:
```typescript
class A {
    constructor(private errors: ValidationError[]) {}
}

// is equivalent to:
class A {
    private errors: ValidationError[];
    
    constructor(errors: ValidationError[]) {
        this.errors = errors;
    }
}
```

When we extend a base class, we have to call super to invoke the constructor of the base class and pass necessary args for constructing that base class to super() .

**Note:** When we're using TS and extending a **BUILT-IN** class to the language, we need to write this line after calling super():
```ts
// only because we're extending a built in class:
Object.setPrototypeOf(this, <current class>.prototype);
```
This is just some behind the scenes stuff that gets our new custom class that is extending the base class to work correctly(the reason is that
we're extending a built-in class).

We're only using a string to communicate the reason of error(the only thing we care is a string) in the DatabaseConnectionError, so we COULD not create it and instead
only use the builtin Error. Tutor just wanted to give us kind of a counterpoint.

So now use those custom errors instead of the generic builtin Error in the request handlers.

So the step 1 of error handling is done which is we're throwing the custom errors out of the request handlers and they're gonna eventually show up inside
of our error handling middleware and there we inspect the incoming error and check to see if it is an instance of those custom errors and send back an appropriate
response.

## 140-013 Determining Error Type:
By checking the err inside the error handling middleware with instanceof, that means that we can tell our error handling middleware how to run some custom logic depending
upon the type of error that it is receiving.

## 141-014 Converting Errors to Responses:
Our error handling middleware is now aware of the type of error that it is dealing with. So now we need to inspect each of those different errors, 
somehow formulate a response and send it back to the user.

The entire reason we got into this discussion was we wanted to make sure that all those different services and all those different kinds of errors are
always gonna result in the exact same structure of error.

So we don't want sth like the 141-014-1 diagram!

The 141-014-2 diagram shows the common response structure for all errors we have inside our app. So no matter the server that it is coming from, we're ALWAYS
gonna send back an object that has an errors property and inside there, it's gonna be an array of objects and each of those objects will have a message and optionally a 
field that refers to what this error is tied to?

## 142-015 Moving Logic Into Errors:
There's sth not so great about current implementation of our error handling middleware:
Right now, we're throwing our RequestValidationError and the DatabaseConnectionError into that error handling middleawre and we've encoded inside there some
very intricate knowledge of exactly how to extract info from every kind of error that exists inside our app(at present there are 2 kinds of errors in addition to
the case where there is some not known error).

Let's imagine we have 10 kinds of custom errors for everything that can go wrong. We need to handle all of them in error handling middleware! It needs to have
some logic that understand every possible error and understand how to extract info from them to send down response to user which all of the responses follow 
the same kind of common response structure.

How we're gonna fix this?
We're gonna inverse or take the opposite of this relationship. In our custom error classes, we're gonna add in a method to each of them called 
serializeErrors. The goal of this method is to take all the info about the error and we're gonna return some array of objects that follow that kind of 
common error structure for our responses. We also make sure that all of those different errors list a status code to send to user as well. 

Once we do this, our error handling middleware is no longer going have to understand how to precisely parse and somehow interpret a custom error. Instead,
the error handling middleware can just verify to make sure that the incoming error is one of those custom error types. Then it can that serializeErrors method.

Look at 142-015-1.

Now the error handling middleware is not gonna grow in much complexity, because no matter what the error is, we're always gonna refer to the same kind of 
serializeErrors method and the same status code from the related error.

By adding the error status code inside the custom error classes, now our error handling middleware doesn't need to know about the precise statusCode to use for
any kind of error, instead, it just knows to take a look at the error's statusCode property.


## 143-016 Verifying Our Custom Errors:

## 144-017 Final Error Related Code:

## 145-018 How to Define New Custom Errors:

## 146-019 Uh Oh... Async Error Handling: