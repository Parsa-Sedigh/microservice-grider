# 11 - Integrating a Server-Side-Rendered React App

## 206-001 Starting the React App

## 207-002 Reminder on Server Side Rendering
This is how a normal react app works in a development setting:
![img.png](../img/section-11/207-002-1.png)

The very first req is gonna fetch us an HTML file that is pretty much empty except for some very basic script tags inside of it.
The browser will load that html file, it's gonna see that some script tags are listed there and so browser will make a followup req(possibly several reqs)
to get some additional JS files. That req(s) will be made to the create-react-app development server that's going to respond with the JS files that the browser
is asking for. The browser will then execute all that JS code, our react app will start up and the react app will (possibly) decide that ift needs to make some followup reqs
to fetch some data.

Instead of this approach, we're gonna use server side rendering:
![img.png](../img/section-11/207-002-2.png)
In a server side rendering approach, browser is gonna make a req to nextjs development server. The nextjs development server is then gonna internally make a couple of reqs off
to different services to fetch some data. Then, the nextjs development server is gonna take that data and build out a full HTML document, a document that has a ton of content rendered into it,
it's then gonna send that html back to the user's browser, all in one single req.

Why use SSR?
- user's gonna see some content appear on the screen much more quickly
- particular if he's on a mobile device, user can see the content faster(on mobile, loading all that JS gonna take more time, but with SSR, user cqn see some content faster)
- SSR apps have better SEO 


## 208-003 Suggestion Regarding a Default Export Warning


## 209-004 Basics of Next JS
Currently, we're starting up the nextjs project **outside** of our k8s cluster, but we're gonna fix this in future.
![img.png](../img/section-11/209-004-1.png)

We want to run our nextjs app inside of our k8s cluster. To do so, we need to make sure that we can build an image out of that client project. We're gonna
set up a deployment file inside of infra/k8s and ... .

Anytime we want to build an image, we create a Dockerfile.

Now just to make sure that we wrote out the Dockerfile and the .dockerignore file correctly, we would like to try to build out that image one time on our local machine manually.
If you're using k8s through goggle cloud, you do not need to do this little build test that we're going to run. Otherwise, if you're running k8s locally, stop the nextjs dev server and
run this command in the directory where Dockerfile exists:
```shell
docker build -t <your docker id>/client . TODO
```

We would need a config file to deploy this nextjs app.




## 210-005 Building a Next Image

## 211-006 Running Next in Kubernetes
Now that the image is built, push it to docker hub. If you're using k8s on google cloud, you do not have to run this command:
```shell
docker push <name of the docker image> TODO
```

We want to get our client running inside of our k8s cluster. There are a couple of things we have to do to get that happening:
1) We need to create a deployment config file inside of oru k8s directory
2) add a config entry to skaffold.yaml file particularly inside that artifacts section(the artifacts section sets up some code syncing). It's gonna make sure
anytime we make a change to some JS files inside of our client directory, those changes get synced into the pod that is running our client image.
4) we want to be able to access this nextjs app from inside of our browser. So we need to somehow make reqs from outside of our cluster and get them
correctly routed inside of it. Remember that we're using ingress nginx for that. In ingress-srv.yaml , we're listing out all of our config to specify routing rules
inside the cluster for incoming reqs from the outside world.

Create client-depl.yaml

Note:
```yaml
  selector:
    matchLabels:
      app: client
```
and this part(inside of `template` section)
```yaml
    metadata:
      labels:
        app: client
```
these 2 sections are how the deployment is gonna find the set of pods that is supposed to manage.

if you're using google cloud for your k8s cluster, you're not gonna put parsa7899/client format of image in `image` attr of `spec.containers` . Instead, you're gonna put
`gcr.io....`

After creating the deployment, we also need a service to allow reqs  to get access to that pod.
Create the config for that service in the same file(client-depl.yaml) after a `---`. 

```yaml
  selector:
    app: client
```
is how the service is gonna find the pods that it's supposed to govern req to.

After we save that new file, skaffold is gonna detect that we've added a new config file to our k8s directory.

After you apply or actually after skaffold applies this new file, you might see an error that says: Cannot find image, your docker id ... . If you see sth like that,
chances are you have not finished pushing your image to docker hub. To fix this, stop skaffold and rerun it after you have pushed the image to docker hub.

Now inside of our skaffold.yaml , we have to setup file syncing by listing out our client project as a build artifact.

Then, we need to update ingress-srv.yaml and add a new path that users can visit and have all reqs that go to some particular path, routed off to our client service.

Currently, inside paths attr of ingress-srv.yaml, we're listing out an array of different paths. Whenever a req comes into ingress nginx, that's gonna attempt to match
the path of that incoming req to the paths we've listed out inside of paths attr of ingress-srv.yaml, **in order**.

The path of our client app in ingress-srv is a **catch-all** path. So we have to always first list out our particular APIs for our different services and then only if we do not 
capture a req against any of the services paths, we want to eventually it to go to the client service.
So list the client service at the very bottom of paths entries.

Now in skaffold logs, you should see some outputs from the client pod:

![img.png](../img/section-11/211-006-1.png)
If you see an error message, rerun skaffold.

Now if you go to ticketing.dev , you should see the nextjs app.

If you got a security warning, we want to bypass that, you probably would type on your keyboard in that site: **thisisunsafe** and it should bypass the https error.

## 212-007 Note on File Change Detection
![img.png](../img/section-11/212-007-2.png)
![img.png](../img/section-11/212-007-3.png)

## 213-008 Adding Global CSS
Global css files have to be wired uo to nextjs in a particular fashion. For this, create _app.js in pages directory and at the top of that file, import the global css files.

**We can only import global css files that must be included in every page, in the _app.js file.**

Nextjs wraps page components inside _app.js component. We can customize that _app.js component.

When we install a package, package.json changes but skaffold doesn't sync changes to that package.json, instead, anytime skaffold detects a change to it, it's gonna attempt to
rebuild the image and deploy it to cluster. So we should(in theory) be able to go to browser, reload the page and see bootstrap kicking in by seeing the font of text changes.

## 214-009 Adding a Sign Up Form

## 215-010 Handling Email and Password Inputs
Whenever ingress nginx routes the req, it goes off to a cluster IP service running inside of our cluster and from the cluster IP service, then the req finally get sent off to
the pod that is running our service container and then inside that container is the express app that's gonna route the req to a req handler where we create our user.
## 216-011 Successful Account Signup

## 217-012 Handling Validation Errors

## 218-013 The useRequest Hook
![img.png](../img/section-11/218-013-1.png)

## 210-014 Using the useRequest Hook

## 220-015 An onSuccess Callback
## 221-016 Overview on Server Side Rendering
![img.png](../img/section-11/221-016-1.png)

We cannot do any data fetching inside of page components themselves when we render a comp with nextjs during SSR phase, we don't get any opportunity to make
reqs inside the comps. Because all of our components are executed or rendered just one single time, so we can not attempt to make req and then wait for the response
and try to update some state or ... . Again because each of our comps during server side rendering are **rendered once**. OFC after our app shows up on the browser,
the getIInitialProps is not used anymore, instead we rely upon the component function itself.

We can use getServerSideProps too!!! I think the course is old!

## 222-017 A note about ECONNREFUSED errors
In the upcoming lecture at about the 4:10 timestamp, we will be moving the axios request from the getInitialProps function directly to the 
LandingPage as part of an explanation. This will likely fail with a long ECONNREFUSED error in your Skaffold output.

Node Alpine Docker images are now likely using the v16 version of Node, so, we will again encounter a situation that will require a catch block.

Change this code in client/pages/index.js

const LandingPage = ({ currentUser }) => {
console.log(currentUser);
axios.get('/api/users/currentuser');

return <h1>Landing Page</h1>;
};
to this:

const LandingPage = ({ currentUser }) => {
console.log(currentUser);
axios.get('/api/users/currentuser').catch((err) => {
console.log(err.message);
});

return <h1>Landing Page</h1>;
};

## 223-018 Fetching Data During SSR
Hooks are used inside comps and not sth like getInitialProps or ... .

Currently, if you make the req on server of nextjs, it fails with ECONNREFUSED, but if we make the req in browser, it works fine. Why?

## 224-019 Why the Error_
This issue is pretty much why tutor wanted to use SSR in the first place.

This diagram shows what happens when we made that req from the browser successfully:
![img.png](../img/section-11/224-019-1.png)

From top left, we have set up our `hosts` file to translate ticketing.dev into localhost(`127.0.0.1:80`). So when we made this req from our browser on ticketing.dev ,
some networking layer on your machine, translated it into that localhost and by default, because we didn't set a port when on ticketing.dev, the port is set to 80.
So that is where we make the req ultimately. Now that port(80) on localhost, has been bound to by ingress nginx. Anytime you make a req to that IP with that port(`127.0.0.1:80`),
ingress nginx is gonna receive that req and route it off appropriately.

The browser will get the html file and eventually in our react app, we have code that makes some followup reqs.

Whenever we try to make a req and we do not specify the domain, by default, your browser is gonna assume that you're trying to make a req to the current domain.

Now with the similar diagram but we're making the req from the server instead of browser(in this one, we're assuming we're not making a req in client side):


![img.png](../img/section-11/224-019-2.png)

In getInitialProps, we again didn't specify a domain for our req. Now in that networking box, we have node's http layer which works similar to browser. if you do not specify
a domain in the req, then node's http layer is gonna assume that you're trying to make a req on your local machine. So node's http layer is gonna automatically stick a domain
to req url of localhost or 127.0.0.1 with port of 80. Now, we're running our nextjs app inside of a container and that container is it's own little world. So when
we tried to make req to 127.0.0.1:80, that req went to port 80 **inside of the container**, it did not somehow get redirected back out to ingress nginx, instead, it went to
localhost inside of that container(client container). There is nothing running on port 80 inside that client container currently. That is why we get 
a `ECONNREFUSED 127.0.0.1:80` error. So the req went to **localhost of that container(that container is a little world, so it has a localhost itself)** and not the auth service.

![img.png](../img/section-11/224-019-3.png)


## 225-020 Two Possible Solutions
![img.png](../img/section-11/225-020-1.png)

We're gonna configure how axios differently depending on whether or not we're making a req from the browser or on the nextjs's server.

There are two different ways of solving this when we were making the req from inside of nextjs:
Option 2: If we're running the code for making req during SSR, put the domain of that auth service that we're trying to access to. We can tell nextjs during SSR
to make the req directly to our auth service. This is not a good option because this implies that our react client code is gonna know the exact service name for
every different thing it's ever gonna want to reach out to. So if we start to introduce other services, we're gonna have to encode those exact service names into our react app
and that;s not good. The other problem is that we need to somehow know those service names, but also which route corresponds to which service? For example http://x-srv/api/xy or ... .

Fortunately all the info has already been encoded in another location, we've already set up all that info inside of ingress nginx. So rather than having option #2 and having our client reach
directly out to the service, we're gonna go with option #1. In this option, we're gonna say that anytime we need to fetch data fro ma service inside of our k8s cluster, we're gonna
have our nextjs application reach out to ingress nginx which is already running inside the cluster. Ingress nginx can then figure out where to send this req off to based upon
just the path(after domain) by itself.

The challenge in this option is what domain are we making the req to? How do we reach out to ingress nginx, while we're inside some pod?

We can reach out to ingress nginx from our local machine by just making a req to localhost:80 or as we have set it up, ticketing.dev . That's gonna forward the req on to ingress nginx.
But how do we do that same kind of thing when we're **already inside the cluster?** We need to figure out how to make a req directly to ingress nginx when we're inside the
cluster.

There is another challenge too. Remember our entire auth mechanism works based upon cookies. On the server in nextjs, when we make a req to our backend, we don't have access
to the browser to automatically manage that cookie. So in nextjs SSR, we need ot make sure that we make the req from nextjs server with the cookie of the original incoming req.

![img.png](../img/section-11/225-020-2.png)
![img.png](../img/section-11/225-020-3.png)

## 226-021 Cross Namespace Service Communication
If our code is running inside the browser, we're just gonna let everything behave as it's currently working. We only have to be concerned about the case
in which we're trying to make a req in nextjs server to send a req.

So whenever we need to send a req while we're doing server side rendering, we need to somehow reach out to ingress nginx and allow ingress nginx to route off this req.

The issue is: How do we actually make a req to ingress nginx?
 
We can access ingress nginx on our local machine by navigating to localhost:80 . But we're trying to make a req to ingress nginx from a totally different
environment which is from inside of that nextjs pod(client pod). If we tried to access localhost inside that pod, then we're gonna access localhost inside the container itself.
That's what resulting in that ECONNREFUSED error. What we should write as the url for making a req in that pod?

Whenever we have two pods that need to communicate with each other in a synchronized fashion, we can do so by using those cluster IP services. For example if we wanted our
client pod directly to that auth pod, we would write out the name of the cluster IP service, so it would be sth like: `http://auth-srv` . Now unfortunately, this rule only works
when these services(in this case client and auth), are inside the same namespace.

What is a namespace?

All the different objects in k8s that we create, are created under a specific namespace. We use namespaces for organizing objects. Till now, we've been working under
a namespace called `default`. If you do:
```shell
kubectl get namespace
```
![img.png](../img/section-11/226-021-1.png)
![img.png](../img/section-11/226-021-2.png)

There's another namespace called ingress-nginx and that's where all of our ingress nginx related stuff works.

![img.png](../img/section-11/226-021-3.png)

Unfortunately, we can't use this style of communication to reach out to a service that exists inside of another namespace. So we can't use that `http://auth-srv` form to
reach out to ingress nginx by saying: http://ingress-nginx-srv . The domain we need to type is more complciated. So let's take a look how we would get our client service to
communicate directly over to the ingress nginx service.

![img.png](../img/section-11/226-021-4.png)

If we want to do some cross-namespace communication to a service created in a different namespace, the full domain for it would be:
`http://NAME_OF_SERVICE.NAMESPACE.svc.cluster.local`

To get NAME_OF_SERVICE, we need to list out all the services that exist inside that namespace. Right now when we run: `kubectl get services`,
it's only listing services from the `default` namespace, but technically there are other services that exist in those other namespaces.

To figure out what services exist inside a namespace:
```shell
kubectl get services -n <name of the namespace>
```
![img.png](../img/section-11/226-021-5.png)
![img.png](../img/section-11/226-021-6.png)

So we need to pass along the cookie from client service to ingress-nginx-controller service and the url would be:
`http://ingress-nginx-controller.svc.cluster.local/api/users/currentuser`

There is a solution for that long url that will allow us to get a similar looking url like: `http://ingress-nginx-controller-srv`(wjhich didn't work for cross-namespace communication).
For this, we need to create sth that is called an **external name service**.

![img.png](../img/section-11/226-021-7.png)

External name service remaps the domain of the req. So it's a nice to have thing. We're not gonna create it in this course! It is sth you could do if you did not want to memorize
that long url.



## 227-022 When is GetInitialProps Called
Here's our plan:

If we're making reqs from the browser, then we're gonna allow the browser to work as usual when it comes to figuring out what domain to make the req to.
For this, we're gonna use a baseUrl of empty string in axios

But if we're making a req during the server side rendering, then we need to set a base url on that req to reach out to ingress nginx directly.

Now how do we know when a req is gonna be executed in the browser vs nextjs server?



![img.png](../img/section-11/227-022-1.png)
![img.png](../img/section-11/227-022-2.png)

When does getIInitialProps gonna be executed inside the browser?

![img.png](../img/section-11/227-022-3.png)


## 228-023 On the Server or the Browser

![img.png](../img/section-11/228-023-1.png)
![img.png](../img/section-11/228-023-2.png)

## 229-024 Ingress-Nginx Namespace and Service - Important Update
updated 6-10-2021

In the upcoming lecture, we will be adding the ingress-nginx service name and namespace to our axios request. This is more or less a reminder to
run `kubectl get services -n ingress-nginx` as shown in the previous lecture "Cross Namespace Service Communication" to find the correct service name for your specific Kubernetes provider.

At the time of writing, the service name for all platforms (Windows, Mac, Linux) and Docker clients (Docker Desktop and Minikube) should be:
**ingress-nginx-controller**

## 230-025 Specifying the Host
To get all of the namespaces:
```shell
kubectl get namespace
```

Whenever a req comes into ingres nginx, ingress nginx needs to know about the host that we're trying to reach(are we trying to reach out to tikceting.dev?).

But when we're making req in this style of url: `???` there's nothing inside of this url to indicate what domain we're trying to access on ingress nginx and ingress nginx **won't**
assume that we're trying to reach to the only host that is listed in it's config(in case there is only 1 host configured there) and you would get a 404.

How do we fix this?

As a second arg to axios.get, use `headers` object and there specify `HOST` property with the domain as it's value. That Host header is gonna be used by ingress nginx.
With this, ingress nginx gonna use it's routing rules for the specified domain.

Now we need to take the cookie from the incoming req that is coming to nextjs server and attach it onto the req that goes over ultimately to the related service in backend.

## 231-026 Passing Through the Cookies
## 232-027 A Reusable API Client
## 233-028 Content on the Landing Page
## 233-029 The Sign In Form
## 234-030 A Reusable Header
## 235-031 Moving GetInitialProps
## 236-032 Issues with Custom App GetInitialProps
## 237-033 Handling Multiple GetInitialProps
## 238-034 Passing Props Through
## 239-035 Building the Header
## 240-036 Conditionally Showing Links
## 241-037 Signing Out
## 242-038 React App Catchup