# Fundamentals

What is microservices? A microservice contains all the code that is required to make one feature works correctly.

Two facts about microservices:

- Each service gets its own database (if needed). Because:
  - We want each service to run independently of other services:
    - to defeat single-point-of-failure
    - to scale more easily
  - Database schema/structure might change unexpectedly.
  - Some services might function more efficiently with different types of DBs (sql vs nosql).
- Services will never, ever reach into another service database.

Data management in monolithic/microservices apps:

![](/md/1.jpg)
![](/md/2.jpg)

In microservices, We can have **sync** (direct) communications between services in which services communicate with each other using direct requests:

![](/md/3.jpg)

Pros:

- Easy to understand.
- Service won't need a database.

Cons:

- Introduces a dependency between services.
- If any inter-service request fails, the overall request fails.
- The entire request is only as fast as the lowest request.
- Can easily introduce webs of requests.

In microservices, We can have **async** communications between services, in which services communicates with each other using events:

- A request comes to a service.
- Service does something locally with the request.
- If the service changes any data, it is going to emit an event to a event broker (bus).
- Other services listens for the events to come in and will populate their databases accordingly to answer some specific queries.

![](/md/4.jpg)

Pros:

- Service D has zero dependency on their services.
- Service D will be extremely fast.

Cons:

- Data duplication. Cost is not an issue though.
- Harder to understand.

# Event Bus (Message Broker)

- For now (you shouldn't do that in reality), we will create one separate service for each resource in our app.
- There are many different implementations of **event bus** such as RabbitMQ, Kafka, NATS, ...
- There are many different subtle features that make async communication way easier or way harder with different implementations.
- We can create our own event broker by creating an express app that has a route (POST /events) which receives the event and fans out (POST) the messages to all services on their (/events) route.
- That event broker should also store all the events so if a service temporarily shuts down or a new service is introduced, that service can ask for all the events to catch up.
- An event can be in any form of data (I guess message would be a better term).

# Docker

- Each service will be run in a separate docker container (which is an isolated computing environment). If we want multiple copies of one service, we can start-up a new container. These containers can be in one machine or different machines.
- Docker solves two problem:
  - how to start a program (e.g we don't need to know `npm start`).
  - all the dependencies that are needed to run a program.

# Dockerizing an app

- We create a `Dockerfile` (with no extension) in the root of our app to create an image out of it. Also create a `.dockerignore` file with the content of `node_modules` so that directory won't copy over:

```Dockerfile
# Specify base image
FROM node:alpine

# Set the working directory to '/app' in the container. All following commands will be issued relative to this dir.
WORKDIR /app
# Copy over only the package.json file
COPY package.json ./
# Install all the dependencies
RUN npm install
# Copy over all of our remaining source
COPY ./ ./

# Set the command to run when the container starts up
CMD ["npm", "start"]
```

- In the terminal, go to the app directory and run `docker build .`. At the end it will give you an image id like eb521ea6b7e6.
- We can run a container out of that image with `docker run eb521ea6b7e6`.

# Some docker commands review

- `docker build -t johndoe/posts .` to build an image based on the dockerfile in the current directory and tag it as `johndoe/posts`.
- `docker run [image id or image tag]` to create and start a container based on the provided image id or tag.
- `docker run -it [image id or image tag] [cmd]` to create and start a container and also override the default command (e.g you can open a shell by sh).
- `docker exec -it [container id] [cmd]` to execute the given command in a running container.
- `docker logs [container id]` to print out logs from the given container.
- `docker ps` to print out info about all the running containers.

# kubernetes

- is a tool for running a bunch of different containers together. We give it some configuration files to describe how we want our containers to run and interact with each other. K8 creates those containers and is going to handle communications between them.
- With K8, we create something called **cluster**:
  - A cluster is a set of virtual machines (which are referred to as **nodes**). Each node will execute some numbers of containers for us. A cluster can have one or thousands of nodes.
  - A cluster has a also a main program to manage nodes in the cluster (called **master**).
  - When we give K8 master a config file to run a program, K8 creates a container or number of containers (depending on the config file) and each container will be randomly assigned to one node to be executed. Each container is inside something called **pod**. We can use `pod` and `container` interchangeably in the context of this course. But know that pod can have multiple containers.
  - To manage the pods of the same nature (identical pods that run containers from the same image), the K8 creates something called **deployment**. The deployment will read the config file and make sure that there are right number of pods up and running.
  - With K8, we define a common communication channel (called **service**) for pods of the same nature that every other containers (e.g event bus) can reach to and send their requests to them. So services are abstracting all the difficulties of trying to figure out what ips or ports some given programs are running on.
- So in short, with K8 creation (lunching and scaling) and communication of services (be aware that kubernetes services are different from services here. K8 services are not running programs.) is really easy.
- To setup K8 in Mac or Widnows, just go to settings (or preferences), then Kubernetes and then enable it.
- To check that it has been installed correctly, run `kubectl version`.
- By default when you start K8 on your local machine, it has one node.

![](/md/5.jpg)

## Config files

- YAML file that tells K8 about the different pods, deployments, and services (referred to as **Objects**) that we want to create.
- Always store these files with our project source code - they are documentations.
- We can create objects without config files (by using direct K8 commands). Do not do this in production environment. Config files provide a precise definition of what your cluster is running.

### Pod

- Create a folder in the root of the application called `infra` for infrastructure.
- Inside that, create a folder `k8s` to host all K8 configuration files.
- Inside that, create a file `posts.yml`:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: posts
spec:
  containers:
    - name: posts
      image: johndoe/posts/0.0.1
```

![](/md/6.jpg)

- To feed the config file to K8, go to the `infra/k8s` folder and run `kubectl apply -f posts.yml`.
- In order to inspect the running pods inside the cluster, run `kubectl get pods`.

![](/md/7.jpg)

### Deployment

- We don't create pods directly (with pod config files). We create a deployment to manage the pods of the same nature. It will really helps us for updating pods of a deployment.
- Inside `infra/k8s`, create a file `posts-depl.yml`:

```yml
apiVersion: apps/v1 # deployment is inside this bucket. Different buckets have different objects.
kind: Deployment
metadata:
  name: posts-depl
spec:
  replicas: 1
  selector: # find all the pods with the label of app: posts
    matchLabels:
      app: posts
  template: # this is where we define the pods
    metadata:
      labels:
        app: posts # we give it this label so it will be managed by this deployment
    spec:
      containers:
        - name: posts
          image: johndoe/posts:0.0.1 # if we omit the version, it will reach out to docker hub.
```

![](/md/8.jpg)

- If you delete a pod inside a deployment by `kubectl delete pod post-depl-7ff956763b`, the deployment will replace it with a new pod.
- If you delete a deployment, all the associated pods will be deleted as well.
- To update an image used by a deployment:
  1. The deployment config file must be using the `latest` tag in the pod spec section (or just omit it).
  2. Make an update to your code.
  3. Build the image with `docker build -t johndoe/posts .`. So no versioning is required anymore.
  4. Push the image to docker hub by `docker push johndoe/posts`.
  5. Run the command `kubectl rollout restart deployment [deployment_name]`.

### Service

- Services provide networking between pods and/or from the outside world to a pod.
- Types of services (There are 4 types but only cluster IP and load balancer is what we use on a daily basis):
  1. `Cluster IP`: Sets up an easy-to-remember URL to access a pod. Only exposes pods in the K8 cluster. They usually have a 1-to-1 relationship with a deployment. So usually, with every service (in the context of microservices), we define a config file for the deployment and in the same file, we write the configuration to create a cluster IP service after `---`.
  2. `Node Port`: Makes a pod accessible from outside world. Usually only used for dev purposes.
  3. `Load Balancer`: Makes a pod accessible from outside world. This is the right way to expose a pod to the outside world.
  4. `External Name`: Don't worry about this one.
- In `infra/k8s`, create a `posts-srv.yml` to create a node port service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: posts-srv # this is the name that will be printed when running: kubectl get services
spec:
  type: NodePort
  selector: # find all the pods with the label of app: posts and expose them to the outside world
    app: posts # this is like a class for html elements
  ports: # this is where we define the ports
    - name: posts # this name can be anything, we can name it after our service.
      protocol: TCP
      port: 4000 # this is the port that the service is running on in the node
      targetPort: 4000 # this is the port that the program is running on in the pod
      # most of the time the above two ports are identical.
```

![](/md/9.jpg)

- To feed the config file to K8, go to the `infra/k8s` folder and run `kubectl apply -f posts-srv.yml`.
- In order to list out the running services inside the cluster, run `kubectl get services`. This command will print out the randomly-assigned port (nodePort 3xxxx). In Mac/Windows, by `http://localhost:3xxxx/posts`, we can reach out to that pod from the browser or Postman. If you are using Docker Toolbox with Minikube, instead of localhost, enter the ip returned from executing `minikube ip` command.
- We could also run `kubectl describe service posts-srv` to see the NodePort.

So in general, we

1. tweak the code so that each microservice can reach another with the name of the cluster ip service and its port: `http://event-bus-srv:4005/events`.
2. build an image for every microservice: `docker build -t johndoe/event-bus .`
3. push the image to Docker hub: `docker push johndoe/event-bus`
4. create a deployment and a cluster ip in one file for each microservice (event-bus-depl.yml):

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: event-bus-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: event-bus
  template:
    metadata:
      labels:
        app: event-bus
    spec:
      containers:
        - name: event-bus
          image: johndoe/event-bus
---
apiVersion: v1
kind: Service
metadata:
  name: event-bus-srv
spec:
  type: ClusterIP # this is totally optional. The default is cluster IP. So remove it.
  selector:
    app: event-bus
  ports:
    - name: event-bus
      protocol: TCP
      port: 4005
      targetPort: 4005
```

5. run all the config files at once by in the `k8s` directory: `kubectl apply -f .` or if it is just an update run `kubectl rollout restart deployment event-bus-depl`.

- For the React app, we will create a docker container and a pod in our cluster but note that when we navigate to it, it serves HTML, CSS, and JS and the browser will issue other requests to other microservices (not the react app).
- We could create a node port for each microservice (bad practice; because the port is random), or a `load balancer service` as a single entry point into our cluster to route requests to appropriate pods (cluster ips tied to pods of the same nature).
- To explain more, the `load balancer service` tells K8 cluster to reach out to its provider (AWS, GC, Azure) and ask for a `load balancer` which lives outside of the cluster. That load balancer takes the traffic and sends it into the cluster (to the `ingress controller` pod running inside the cluster). Then, the ingress controller with a set of routing rules decides to route the traffic to other services inside our cluster.
- To create a load balancer service + an ingress, we will use `ingress-nginx` project (do not mix it up with `kubernetes-ingress` project which does a similar thing). The installation process is running:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
```

to install an ingress-nginx-controller and a load balancer service.

- Now, we have to give the ingress controller the routing rules. The problem is that it doesn't do the routing based on the HTTP request type and it only looks at the path. So we have to change the path for creating a post (POST to /posts in posts service) and listing posts (GET /posts in Query service) different (instead of POST /posts we change it to /posts/create in posts service and also in react app and build, push, and update the deployment). So we create a `ingress-srv.yml` inside `k8s`:

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-srv
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: "true" # to use regex in out path
spec:
  rules:
    - host: my-app.com # with K8, we can have multiple apps with different domains. Here we have just one app, so we give it a domain name of for example my-app.com and in Windows, in `C:\Windows\System32\Drivers\etc\hosts`, we add `127.0.0.1 my-app.com` to redirect us to localhost. Then in the browser or postman, we hit `my-app.com/posts` to list the posts. For the prod env., the host will be actual server address.
      http:
        paths:
          - path: /posts/create
            backend:
              serviceName: posts-clusterip-srv
              servicePort: 4000
          - path: /posts
            backend:
              serviceName: query-srv
              servicePort: 4002
          - path: /posts/?(.*)/comments # use of regex in our path
            backend:
              serviceName: comments-srv
              servicePort: 4001
          - path: /?(.*) # for react (when we have react-router in a SPA approach) it should be at the very end (because matching is from top to bottom)
            backend:
              serviceName: client-srv
              servicePort: 3000
```

- So in react app, we will change all the requests to different services (which are on different ports) to only my-app.com (instead of localhost:4xxx). Then, build the image and push it to docker hub. After that, create a deployment and cluster IP for it. Finally, apply it.

## Skaffold

- In the DEV env, currently, the process of updating each microservice (build image -> push to docker hub -> run `kubectl rollout restart deployment [depl_name]`) is a real pain (but it is how we do it in prod).
- Skaffold makes it easy to update code in DEV env.
- Skaffold also makes it easy to create/delete all objects of a project (for one cluster) at once. So you can work on different projects in your local machine (because it is just one cluster).
- Install Skaffold from `skaffold.dev` website (in windows you have to install chocolatey first).
- Create a `skaffold.yml` file in the root of the project (not in `infra/k8s`).

```yml
apiVersion: skaffold/v2alpha3
kind: Config
deploy:
  kubectl:
    manifests:
      - ./infra/k8s/* # this does 3 things: when we strat skaffold, apply all the files in that folder. When we make a change, apply these files. And when we stop skaffold, delete the objects associated to these config files.
build:
  local:
    push: false # by default whenever skaffold makes a change to one of the images, it tries to push it to docker hub which is not needed.
  artifacts:
    - image: johndoe/client
      context: client # it means that for the above image, look into the client folder inside our project and update our pod if anything changes. It has two ways.
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "src/**/*.js" # if a js file changes, skaffold directly updates our pod. For other files, skaffold will build the entire image and throw it to update the pod (no need to push to docker hub).
            dest: .
    - image: johndoe/comments
      context: comments
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "*.js"
            dest: .
    - image: johndoe/event-bus
      context: event-bus
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "*.js"
            dest: .
    - image: johndoe/moderation
      context: moderation
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "*.js"
            dest: .
    - image: johndoe/posts
      context: posts
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "*.js"
            dest: .
    - image: johndoe/query
      context: query
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: "*.js"
            dest: .
```

Run `skaffold dev` in the root directory. With `ctrl + c`, all the objects will be shut down.

# THE PROJECT

## The entities (resources)

![](/md/10.jpg)

## Services

![](/md/11.jpg)

- Now, we are creating a separate service to manage each type of resource (with the exception of expiration service), but feature-based would be a better approach (for example, ticket, order, and expiration could be combined in a single service).

## Events

![](/md/12.jpg)

## Architecture

![](/md/13.jpg)

## Auth service

![](/md/14.jpg)

- Create a package.json by `npm init -y` in the auth folder.
- Install `npm i express @types/express typescript ts-node-dev`. ts-node-dev is like nodemon for TS.
- Run `tsc --init` to create a `tsconfig.json` file in the root of the auth project.
- Now create two folders `src` and `build` in the root of the auth project.
- Create an `index.ts` in the `src` folder.
- Add `"start": "ts-node-dev --poll src/index.ts"` to the script section of the `package.json` file.
- Add `Dockerfile` and `.dockerignore` in the root of the project.
- Build the image by using `docker build -t johndoe/auth .`.
- Create `auth-depl.yml` in `infra/k8s` folder.
- Create a `skaffold.yml` file in the root of the whole project (not in `infra/k8s`).
- Run `skaffold dev`.
- Install `ingress-nginx` and then create `ingress-srv.yml` inside `k8s`.
- Add `127.0.0.1 ticketing.dev` in `C:\Windows\System32\Drivers\etc\hosts` (open the editing program in administrator mode).

## Leveraging a cloud environment for development

- If your computer becomes laggy, do this; otherwise, jump to another section.
- Changing a `.ts` file:

![](/md/15.jpg)

- Changing `package.json` file:

![](/md/16.jpg)

- Go to `https://console.cloud.google.com/` and create a new project `ticketing-dev`.
- Select the project.
- On the LHS, select `Kubernetes Engine` and then select `Clusters`.
- After a while, click on `Create cluster`.
- Change the name of the cluster to `ticketing-dev`.
- Select a zone that is physically closer to you.
- Change the version of K8 to at least 1.15.
- Go to default-pool -> Nodes and change the `Machine type` to `g1-small`.
- Click on Create (this cluster has 3 nodes by default; unlike the local machine that had only 1 node.).
- We have to change the **context** of kubectl so by running it on our local machine it connects to the GCP (for now, the context is `docker-desktop`). To do that, we install Google Cloud SDK.
- Run `gcloud auth login` to login.
- Run `gcloud init`.
- Choose to reconfigure.
- Choose your google acc.
- Choose the `ticketing-dev-234234` project.
- Choose Y for configuring region and zone and select the one that you selected for your project.
- Close docker desktop.
- Run `gcloud components install kubectl` to install kubectl on your machine (not the one from docker).
- Run `gcloud container clusters get-credentials ticketing-dev`.
- To enable google cloud build, on the LHS, go to `Cloud build` and then enable it.
- To update `skaffold.yml` to use google cloud build, add `googleCloudBuild: projectId: ticketing-dev-281906`, comment out the `local` section, change `johndoe` to `us.gcr.io/ticketing-dev-281906` and update the image name as well in `auth-depl.yml`.
- Run `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml` to setup ingress-nginx on our google cloud cluster. To see if that works, go to Networking -> Network services -> Load balancing and we can see that a load balancer is created for us. Click on it to see its ip address.
- To update our hosts file again to point to the remote cluster, put that ip address `35.197.188.104 ticketing.dev` in `C:\Windows\System32\Drivers\etc\hosts` (open the editing program in administrator mode).
- Run `skaffold dev` to restart (if you have some credential problem, run `gcloud auth application-default login`).
- After you finished with development, go to GCP console -> Kubernetes engine -> Clusters and delete the cluster.

## Response Normalization Strategies

![](/md/17.jpg)

- We have to use a consistent error response structure for different microservices so that React app doesn't have to know every response structure.

## Database Management and Modeling

- To create a mongo pod for each microservice, just create a `auth-mongo-depl.yml` in the `infra/k8s` folder for example. Then, restart the skaffold.
- For now, if we delete or restart the pod running MongoDB, we will lose all of the data in it!

## Authentication Strategies and Options

![](/md/18.jpg)

- To tell that if the user is logged-in:
  - One option is that other services call Auth service directly (sync), without using an event bus.
  - The second option is to teach every service how to inspect cookie/jwt and decides if the user is authenticated (the sign-up and sign-in are still responsibility of auth service.). The problem with this approach is that if we update the user status (such as isBanned), other services wouldn't know.
- We are going to go with option number two to have independent services but you certainly can have a hybrid of sync and async communication between your services. In addition you can still solve the issue with async communication by either using a short-lived tokens (and if the token is expired, the microservice will reach the auth service in a sync manner to refresh it or just reject the request) or have the auth service to produce an event whenever a user is banned, then other services can subscribe to that event and persist the banned users' id in a short-lived in-memory cache.
- Cookies are automatically stored and sent by the browse on all follow-up requests but with JWT, we have to send it over in header (Authorization header) or body (token field in the body) or even set as cookie (Cookie header):

![](/md/19.jpg)

- With a normal react app (SPA with CRA (=create react app)), the browser sends req to other services:

![](/md/20.jpg)

- With a SSR (Server Side Rendered) app, the browser sends the request to a client backend and the client backend sends the request to other services. Of course this is a direct call (sync communication). The benefits of SSR are SEO and faster page loading:

![](/md/21.jpg)

- The problem is for the first request, we don't have any JS on the client, how we want to reach to local storage and sends JWT in the header or body? The solution is store the JWT in the cookie! Remember it is not a cookie-based authentication, because we don't encrypt the cookie. We just use cookie to transport JWT.

- In SPA, we should create an Nginx server to serve one index.html and subsequently all the CSS and JS, with SSR, we will have a Next JS server to serve all the pages.

- We have to figure out a way to share JWT secret between different microservices in K8 cluster -> a secret object! The key-value pairs inside this object will be env variable inside the containers of each pod in the cluster. We will run this command imperatively (because we don't want to list our secrets in a config file):

```
kubectl create secret generic jwt-secret --from-literal=jwt=thisIsASecret
```

Then, in the container definition in the auth-depl, we add:

```yml
env:
  - name: JWT_KEY
    valueFrom:
      secretKeyRef:
        name: jwt-secret
        key: jwt
```

## Testing Isolated Microservices

- We test different microservices in isolation. Because, testing the communication and interaction between different services requires creating an environment which is costly and complicated.
- We will write three kind of tests:
  1. Basic request handling: issue a request and see how it is handled (rejected, changes in the db, etc.).
  2. Some tests around the model which will be unit test kind of approach.
  3. Tests about emitting and receiving events (if a microservice emits an event or listens for incoming events and processes it in a particular way.).
- We are going to run these tests directly from our terminal without using docker. So if we run `npm run test`, it is going to start-up our test server in our local machine (so for this simple project and auth service, it needs that node and a in-memory copy of MongoDb are present in our local machine but for more complex projects which might have way more dependencies, it can be hard.).
- The reason for in-memory mongodb is running multiple databases (for different services) at the same time on our local machine.

![](/md/22.jpg)

- For testing purposes, we have to re-structure the express app, so we give the app to supertest without listening on port 3000.

![](/md/23.jpg)

- So now the app file does not start-up the application and just configures it.
- Install other dependencies with `--save-dev` flag such as `@types/jest @types/supertest jest ts-jest supertest mongodb-memory-server`.
- Change the `package.json` file to 1. watch for changes. 2. teach jest to understand TS 3. to run a file after setup the server (to setup mongodb memory server, get mongoose to connect to it and etc.):

```json
"scripts": {
  "start": "ts-node-dev --poll src/index.ts",
  "test": "jest --watchAll --no-cache"
},
"jest": {
  "preset": "ts-jest",
  "testEnvironment": "node",
  "setupFilesAfterEnv": [
    "./src/test/setup.ts"
  ]
},
```

- When we make use of jest, we follow the convention that if we are trying to test a file, we make a folder inside the same directory called `__test__`. Then inside that folder, create a file with the same name but the extension of `.test.ts`.

## Next JS

- Run `npm init -y` inside the client folder.
- Intsall `npm i react react-dom next`.
- Create a `pages` folder in the client project. Next will look at this folder and create the routes based on it (file names map up to routes).
- To run Next project, add `"dev": "next"` in the scripts section of package.json file.
- After running the Next project in K8, add `next.config.js` to ensure hot module reloading for Next (because Next is finiky when it is run in Docker). Make sure that the pod which runs Next is restarted by deleting the pod (the deployment will recreate the pod and Next will load that middleware).
- To add a global css, first we install bootstrap by `npm i bootstrap` and then add `_app.js` file which is a wrapper component for every page (every page will be passed to it as a Component). Next.js uses the App component to initialize pages. You can override it and control the page initialization such as add global css. To override the default App, create the file ./pages/\_app.js. `pageProps` is an object with the initial props that were preloaded for your page by one of Next's data fetching methods, otherwise it's an empty object.
- We will create a custom hook (useRequest) to reuse the code for making different requests and shows the associated errors. This custom hook is a function which takes some inputs, has a function to do the request and a piece of state with some JSX containing errors (null if not any). The hook returns the function and that piece of state in an array (here the object makes more sense).
- To navigate the user programmatically in Next application, we will make use of `import Router from "next/router";` and then `Router.push("/");`.
- For each page, we can create a `getInitialProps` function which will be called whenever Next tries to build HTML on the server for the first time. This is our opportunity to send request to other services and fetch some data. That data will be provided to our component as props. During server side rendering, we can't make requests from the component itself (in hooks) because each component is executed once and the resulted HTML will be sent back to the user during server side rendering. After it shows up on the browser, getInitialProps is not used anymore and the component can make a request from the browser.

```js
const LandingPage = ({ color }) => {
  console.log("I am in the component", color);
  return <h1>Landing</h1>;
};

LandingPage.getInitialProps = () => {
  console.log("I am on the server!");

  return { color: "red" };
};

export default LandingPage;
```

- In getInitialProps we should send the requests to cluster IP services (a better solution is to send the request to ingress-nginx and it redirects it to desired microservice. In this way the React app does not need to know the address for every microservice (cluster IP name). So we need to know how to call the ingress-nginx from inside the cluster. The other challenge is to extract the cookie from the original request and set it on the request made from the node env.); but in the browser because we use ingress-nginx, we issue the request to the same domain (so we use relative URL there).

![](/md/24.jpg)

- Because our objects that we have defined (default namespace) and the ingress-nginx (ingress-nginx namespace) are in different namespaces (we can get different namespaces that we have on our cluster by `kubectl get namespace`), we can't directly communicate by using cluster IP service. Instead we use `NameOfService.Namespace.svc.cluster.local` for the domain. To get the name of service in ingress-nginx namespace we should run `kubectl get services -n ingress-nginx` which is `ingress-nginx-controller`.
- So we should send the request to different domains if it is sent from the browser or the server:

![](/md/25.jpg)

- But be aware that not all requests from getInitialProps are sent from the server! If we redirect from one page to another programmatically, it will be executed on the client:

![](/md/26.jpg)

![](/md/27.jpg)

- So we can write something like (which is nasty):

```js
LandingPage.getInitialProps = async ({ req }) => {
  // whenever getInitialProps called on the server, the first argument is an object which has the original request object.
  // we are interested in cookie header and also the host header which is ticketing.dev.
  if (typeof window === "undefined") {
    // we are on the server.
    const { data } = await axios.get(
      "http://ingress-nginx-controller.ingress-nginx.svc.cluster.local/api/users/currentuser",
      {
        headers: req.headers,
      }
    );

    return data;
  } else {
    // we are on the browser.
    const { data } = await axios.get("/api/users/currentuser");

    return data;
  }
};
```

- We create a reusable function to build the client based on the context and we will call it in getInitialProps that we will be writing and pass the context to it.
- To add a global layout, again the \_app is the way to go.
- If we want to add a reusable menu bar, the menu bar also needs to know about the current user; so we move the getInitialProps logic from landing page to App component (the first issue is that context that the getInitialProps of the app component is called with is a little bit different from other components -> look at the picture below) and pass it as props to both children. If both the \_app and the component have getInitialProps another issue rises (when we tie getInitialProps to app component, the getInitialProps function we tied to an individual page do not get automatically invoked anymore. So we have to invoke that in the getInitialProps function of the app component).

![](/md/28.jpg)

- For sign out, note that the request should come from the component and not getInitialProps; because we want to clear the cookie in the browser (getInitialProps works in both server and client side while componentDidMount is only for the client side).

![](/md/57.jpg)

- To capture the id in Next, we warp the name of the query parameter with square brackets for the filename. Then we can receive the query parameter in our page component (in the `context.query` of getInitialProps). Then we can make a request to get the details of that id in the getInitialProps and then return the results. You may recall that what is returned from getInitialProps, will be merged to the other props of the components.

![](/md/58.jpg)

- Note that if we hadn't implemented the App component, the getInitialProps of each page would have been called automatically.
- For href attribute, we write the path to the file that you want to show. As the second property we provide `as` which is the true real url that we want to navigate to.

```js
<Link href="/tickets/[ticketId]" as={`/tickets/${ticket.id}`}>
  <a>View</a>
</Link>
```

- We will do something similar for pushing to a wild-card route:

```js
Router.push("/orders/[orderId]", `/orders/${order.id}`);
```

- Whenever we return a function from `useEffect`, that function will be invoked whenever we are about to navigate away from this component or if the component is going to be re-rendered (for the second case, it is only going to call that function, if we have a dependency listed inside of the second array argument of useEffect).
- If you write:

```js
<button onClick={doRequest} className="btn btn-primary">
```

The first argument to the doRequest will be the event object and because that function attaches the arument to the request, you have to write it like this:

```js
<button onClick={() => doRequest()} className="btn btn-primary">
```

## Code Sharing and Reuse Between Services

![](/md/29.jpg)

- We will publish all the common code as an NPM package to NPM registry. Then, in each project, we will install it as a dependency. The nice thing is that we can version the code inside this common library and different projects can use different versions. Downside is that every time we want to change our common code, we are going to make our change, push it up to the NPM registry, then update the versions in different projects.
- Regarding the security of the code, we will publish it to an organization registry (public is free but easily can be converted to private if needed). So click on `Add organization` in npm.js.
- After creating the common project and generating the packahe.json, change name to `"name": "@betickets/common",`.
- Create a local git repo and commit it.
- Run `npm publish --access public`. (if you get an error, run `npm login` first.).
- Because there might be differences in out TS settings between the common lib and our services and even the services might not be written with TS at all, our common library will be written as TS and published as JS.
  - So run `tsc --init` and then install `npm i typescript del-cli --save-dev`. del-cli is just for cleaning the build directory before each build to start with a clean slate.
  - Add `"clean": "del-cli ./build/*",` and `"build": "npm run clean && tsc"` in package.json.
  - Uncomment `"declaration": true` and `"outDir": "./",` and change it to `"outDir": "./build",` in `tsconfig.json` file.
  - Change/add
  ```json
  "main": "./build/index.js",
  "types": "./build/index.d.ts",
  "files": [
    "build/**/*"
  ],
  ```
- Add `.gitignore` file.
- Add `"pub": "git add . && git commit -m \"Updated\" && npm version patch && npm run build && npm publish"` in the scripts section. This is bad and you should not usually do this and you have to run the commands manually because of increasing just patch number, generic git commit message and adding all to git.
- Move `errors` and `middlewares` folder from auth project to the common project and in the `index.ts`, add `export * from "./errors/bad-request-error";` for each file.
- Install missing dependencies.
- Install `npm i @betickets/common` in auth project.
- So the flow will be: Update the common -> `npm run pub` in common -> `npm update @betickets/common` in auth.
- To see if that really takes effect: `kubectl get pods` -> `kubectl exec -it auth-depl-76fb9d7c97-ttsps sh` -> `cd node_modules` -> `cd @betickets/common` -> `cat package.json`.

## Ticketing Service

![](/md/30.jpg)

![](/md/31.jpg)

## NATS Streaming Server

- NATS and NATS Streaming Server are two different things (be aware of this, when looking at the docs). NATS SS is built upon NATS and is more advanced which we will be using.
- Write nats-depl.yml file.
- To communicate with NATS, we will use a client library called node-nats-streaming to send off our events and also listen to events in our microservices.
- In NATS SS, there is this concept of channels (or topics or subjects), so that, the microservices emit events to a specific channel and all the microservices subscribed to that channel will receive it. Channels can be called after the event itself (ticketCreated channel).
- For handling outages or in case of new services, we should store the events. NATS SS stores all events in memory (default) but can be configured to store them in flat files or in MySQL/Postgres DBs.
- nats-playground project is run in the local machine and wants to connect to nats ss inside the K8 cluster that we have. We have three options to connect to it.
  1. Add rule to ingress-nginx.
  2. Create a node port service.
  3. `kubectl port-forward nats-depl-5c4c888466-2td4n 4222:4222` forward the local port 4222 to port 4222 in that pod (of course kubectl command-line tool must be configured to communicate with your cluster). As long as this command is running we can communicate.
- Whenever we connect to NATS SS, the server maintains a list of clients that is connected to. So if we want to run multiple clients (copies of listener/publisher), we have to give it a different id. We use a random id and we will run listener multiple times to create different instances of listener.
- Usually, the same instances of one service should not process an identical message that they all receive (for example imagine that all of them want to reach a database and insert something).
- To prevent that, we use `queue group`. We create a queue group inside the channel and all the instances of a same service will connect to that queue group. Each channel can have one or more queue groups. When a message comes in to a channel, NATS SS will randomly sends it off to one member of each queue group inside of that channels. So services can subscribe to the channel or a queue group inside a channel.
- The default behavior is that when a subscription receives a message, but a problem happens during the processing, the message will be lost. So we will pass a third argument (an option object) to the subscribe function to ensure the acknowledgement of the event: `const options = stan.subscriptionOptions().setManualAckMode(true);`. If the message handler, doesn't acknowledge the event (by `msg.ack()`), NATS SS will resend it another member after 30 secs (if it has subscribed directly, it will resend the message).
- To enter the monitoring page, run `kubectl port-forward nats-depl-5c4c888466-2td4n 8222:8222` in a different terminal and let it be open, then navigate to `http://localhost:8222/streaming`. with `http://localhost:8222/streaming/clientsz?subs=1`, you can see more details.
- When a client goes offline, NATS SS won't notice immediately and it might send an event to it. To check the health of clients, NATS SS checks the heartbeat every few seconds which is configurable in nats-delp.yml args section.
- Graceful shutdown: another way of notifying NATS SS is intercepting closing process by adding `process.on("SIGINT", () => stan.close());` and `process.on("SIGTERM", () => stan.close());` and handling close event by the client by adding `stan.on("close", () => {console.log("NATS connection closed"); process.exit();});`. Note that if you kill the process by task manager, that signals won't be sent and NATS SS won't notice and it falls back to heartbeat mechanism.
- Anyhow, there might be cases that NATS SS thinks a client is alive and sends it an event, then figures out it is dead and sends it to other client and in the meantime another event is raised and sent to a live client and the older event is processed after a newer event. Another scenarios:
  - Listeners can fail to process the event.
  - One listener can run more quickly than another.
  - NATS might think a client is alive when it is dead.
  - We might receive the same event twice: one listener can finish a process one moment after finishing the resend duration.
- You should know that these concurrency issues can happen in sync communication of microservices and even in monolith architecture (if we have multiple instances of an app which talks to the same database, one request can be handled quicker than other requests). It is just more prominent in async communication of microservices because of event bus and ...
- You might think that using one copy of each service can solve most of the problems above, but that's not a viable option because we want to scale horizontally. Other solutions: share state between services of last event processed, last event processed tracked by resource id, ...
- All the solutions that rely on NATS and it sequence number to solve the concurrency issues are wrong. We should re-design the app itself. If the order is important in the app we should create a field for it in the publisher database, create a last_txn_number field in the listener database and in the listener check to see if the number of the message is exactly one greater than the last_txn_number for that specific record:

![](/md/32.jpg)

- By chaining on `setDeliverAllAvailable()` option, whenever a listener comes up (or restarts) it receives all the events that have been emitted in the past from NATS on the startup. But we don't want the events that have been already processed by this service so we chain another option `setDurableName("orders-service")`. Just note that this option works the way you think if you accompany it with providing the queue group name argument in the subscribe function.
- We are going to define a Listener and a Publisher abstract class:

![](/md/33.jpg)

- These two base classes along with the enum for list of event names (subjects) and interfaces for different events (which specifies the subject and the structure of data for each event) will be moved to the common module.
- For testing, publishers are very similar to making a network request. So there is not a lot to test.
- On the other hand, listeners are super similar in nature to request handlers! Tons of stuff to test.
- Whenever you think your NATS deployment has a lot of trash data in it, just delete the pod so it will be restarted with zero data.

## Managing NATS Client

- Our tickets app (or any other microservices) needs just one connection to NATS SS (a client) just like mongoose that you call connect method on mongoose object to connect to a MongoDB once and in all the files just use that mongoose client object which is pre-configured. But the difference is that when connecting to NATS it returns the client so if we connect to NATS in index.ts (which return a NATS client or stan) and imports it in other route handlers to feed it to publisher or listener instances, there will be a cycle (route handler imports from index.ts, app.ts imports route handlers, index.ts imports app.ts) so we will create a nats-client.ts file to have a singleton:

![](/md/34.jpg)

- To publish, create an events folder and for each event create a sub-class.
- Then, import it in wherever you want to emit an event, instantiate it and give it the client from that NatsClient class. In new, we awaited the publish but in update we didn't await the publish method. So the question is what if the data is persisted to the db but the publishing fails? So the integrity between different databases will be lost. To solve this issue, we would save the record and the event (with a flag of sent?) related to the record at the same time in another collection (using a transaction/rollback) and we would implement separate code/process to pull events from events collection and publish them to NATS if they are not sent already.
- Currently, some tests are failing if you run `npm run test` because for test, the code in `index.ts` won't run and the natsClient will not be initialized. But in test we don't want NATS service so we will mock NatsClient class. So:
  1. Find the file that we want to mock.
  2. In the same directory, create a folder called `__mocks__`.
  3. In that folder, create a file with an identical name to the file that we want to mock.
  4. Write a fake implementation.
  5. Tell jest to use that fake file in our tests (in all test files -> But because this is not efficient, just insert it in the setup file to be run for all tests).
- Now, we just have to ensure mock invocation. But we can't just set `publish: jest.fn()` because we want to pass it a callback to be executed so we chain `publish: jest.fn().mockImplementation((subject: string, data: string, callback: () => void) => {callback();})`. Now, we can write some assertions that our mock function is actually invoked.

## Cross-Service Data Replication In Action

- Instead of storing ticket price in Order document, we will have another Ticket collection which will be a replication of Ticket collection in tickets service. This replication collection will be updated based on that version and last_version approach (to prevent concurrency and out-of-order issues) for two events of `ticket:created` and `ticket:updated`. Note that MongoDB and Mongoose can manage all of this version stuff for us and we just include them in the event data.

![](/md/35.jpg)

![](/md/36.jpg)

![](/md/37.jpg)

- Note that in orders service, we cannot embed ticket documents in order documents because there are some tickets that are waiting to be ordered (not all tickets have orders yet). So we will use ref/population feature.
- Note that just like the events that should have identical structures between microservice, the order status also needs to be uniform across different service so we define an OrderStatus enum in the common module.

![](/md/38.jpg)

![](/md/39.jpg)

![](/md/40.jpg)

![](/md/41.jpg)

![](/md/42.jpg)

- Record updates with OCC (Optimistic Concurrency Control) in mongoose:

![](/md/43.jpg)

- Install `mongoose-update-if-current` in tickets service.
- Add `import { updateIfCurrentPlugin } from "mongoose-update-if-current";` in ticket model in tickets service.
- After defining ticketSchema, add `ticketSchema.plugin(updateIfCurrentPlugin);`.
- To change `__v` to `version` add `ticketSchema.set("versionKey", 'version');` before the plugin and also add version field to TicketDoc interface.
- What this plugin does is that if two instances of a document exists in memory and we want to save the updates oo both to the database it checks the version with what is in the database and if it is the same, it does the update and increments the version so the second update will fail and throw an error (This plugin brings optimistic concurrency control to Mongoose documents by incrementing document version numbers on each save, and preventing previous versions of a document from being saved over the current version.).
- When should we increment or include the `version` number of a record with an event?
  - Increment and include the `version` number whenever the `primary service responsible for a record` emits an event to describe a `create/update/destroy` to a record.

## Expiration Service

![](/md/44.jpg)

Publishing expiration completed event is an scheduled task. There are four options to do scheduled jobs:

![](/md/45.jpg)

![](/md/46.jpg)

![](/md/47.jpg)

![](/md/48.jpg)

- In expiration project install `npm i bull @types/bull`.
- We don't need `expiration-srv` for expiration deployment because no request is going directly to this service.
- This is how how Bull is traditionally used (a time-consuming request comes to the web server, the web server sends a job (a job is a plain js object) to redis and the worker server polls redis for new jobs (just like the example in Docker section). Bull is present both at the web server and the worker server.). But note that in our app there is no separate worker server and web server and there is only one expiration service.

![](/md/49.jpg)

![](/md/50.jpg)

## Payments Service

![](/md/51.jpg)

![](/md/52.jpg)

- Whenever a user is trying to submit a request to pay for an order, we need to understand what order they are trying to pay for and validate that payment (it is the current user that tries to pay and the payment has correct amount). So we need to replicate the order data into payment's service database (we are only interested in id, status, version, userId, and price).

### Stripe Flow

- The token is some kind of authorization from the user. We get that token and make sure the amount is correct and the user is the right one and the order is still valid and then send it to stripe api ourselves and then we create a charge.

![](/md/53.jpg)

![](/md/54.jpg)

- We also need to install stripe SDK and also sign-up for an API key. We store this API key in a secret object inside our cluster:

```
kubectl create secret generic stripe-secret --from-literal=STRIPE_KEY=sk_test_OKBM67rLAdixSAhvuNohK2qw00GCjdgNGm
```

![](/md/55.jpg)

- An alternative way of testing payment is reaching to actual stripe api. To do this, delete the mock implementation of stripe from `__mocks__` directory and also delete the `jest.mock("../../stripe");` from `new.test.ts` file. But bare in mind that because tests are run in our local machine and not in the cluster, we need to set `STRIPE_KEY` environment variable in our local machine. The other issue is that our route handler does not return anything to see if the charge is successful; but stripe has an endpoint to list out 10 last payments. So in our test we can create a charge with random amount (`const price = Math.floor(Math.random() * 100000);`) and then directly call that endpoint to get 10 recent charges and verify that our random amount is among them. I personally think that it is not a good unit test because it will take time to reach out to the network and mocking is better. We could change the route implementation to send the client the response from stripe but it is not a good practice to change the code for testing.

```js
const stripeCharges = await stripe.charges.list({ limit: 50 });
const stripeCharge = stripeCharges.data.find((charge) => {
  return charge.amount === price * 100;
});

expect(stripeCharge).toBeDefined();
expect(stripeCharge!.currency).toEqual('usd');
```

![](/md/56.jpg)

- The payments collection is to have a record of the charges for each order (to relate charges and orders together). We are not going to use it in our app. We could have a dashboard to show the user the list of his payments.

## CI/CD

- Development workflow:

![](/md/59.jpg)

- We can create single repo for each service or one repo for all services. But the first option has a lot of overheads. You have to create many repos, auth keys, and CI/CD pipelines.
- We will use `Github actions` to run tests for each microservices, deploy code, anything you imagine.

![](/md/60.jpg)

![](/md/61.jpg)

- Delete everything and rename the file to tests.yml:

```yml
name: tests

on: pull_request # everytime a PR created, updated or reopened

jobs: # things that we want to do
  build: # start-up some virtual machines (container)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2 # take all the code
      - run: cd auth && npm install && npm run test:ci
```

![](/md/62.jpg)

- `test:ci` is a script in our auth package.json to run jest only one time without watch mode.
- Now, we can create a PR and see if it works.
- You know that when a PR is open, you can push to that branch and it automatically will be reflected on the PR.
- We will create a different github action file for each service (we could write all our tests in one test.yml file to have them run in series but in parallel is better.)

![](/md/63.jpg)

- If we add:

```yml
on:
  pull_request:
    paths:
      - "orders/**"
```

to each service, only the tests related to services that have been changed inside of this PR.

- Comparing different hosting providers to run a K8 cluster of 3 nodes, each with 2gb of ram and 1 cpu.

![](/md/64.jpg)

![](/md/65.jpg)

![](/md/66.jpg)

- Kubectl decides on how to connect to a given cluster by using context:

![](/md/67.jpg)

- Which goes to https://github.com/digitalocean/doctl. Install doctl. Which is in fact a doctl.exe and is command tool program.
- First generate a token with a name of for example doctl:

![](/md/68.jpg)

- Then run `doctl auth init` and you will be prompted to paste the token from the UI.
- We can use doctl to install a context to connect to our cluster from local kubectl.

![](/md/69.jpg)

- By running `doctl kubernetes cluster kubeconfig save ticketing` the context will be changed to the cluster in DO. So if you run `kubectl get pods` no resources will be found even if you run some on google cloud.
- By running `kubectl config view`, you can list out all the contexts:

![](/md/70.jpg)

- By running `kubectl config use-context gke_ticketing-dev-281906_australia-southeast1-a_ticketing-dev` or `kubectl config use-context docker-desktop`, the context will be changed. You can change the context easily by using docker desktop.
- Change it back to DO. Note that we are not going to use it to deploy to DO. We only use github workflow to do that but this is just for viewing things, debugging, pull logs and ...
- This is the deployment plan, each row is github workflow file which will be run whenever we want to merge to master:

![](/md/71.jpg)

![](/md/72.jpg)

![](/md/73.jpg)

![](/md/74.jpg)

- After merging the PR, you can see that this workflow starts and pushes the code to the docker hub.

![](/md/75.jpg)

- Complete the `deploy-auth.yml`:

```yml
name: deploy-auth

on:
  push:
    branches:
      - master
    paths:
      - "auth/**"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: cd auth && docker build -t johndoe/auth . # docker is installed on ubuntu-latest container
      - run: docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD # we have to tell the container in github our credentials
        env: # secrets are not env variables. So in this section, we will introduce them as env variables.
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
      - run: docker push johndoe/auth
      - uses: digitalocean/action-doctl@v2 # install doctl inside the container
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }} # authorize doctl to connect to DO. You have to create a new access token in DO dashboard with the name of for example github_access_token.
      - run: doctl kubernetes cluster kubeconfig save ticketing # shove the context to kubectl
      - run: kubectl rollout restart deployment auth-depl # kubectl is pre-installed
```

- Create `deploy-manifests.yml`:

```yml
name: deploy-manifests

on:
  push:
    branches:
      - master
    paths:
      - "infra/**"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: digitalocean/action-doctl@v2
        with:
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
      - run: doctl kubernetes cluster kubeconfig save ticketing
      - run: kubectl apply -f infra/k8s
```

- But remember that `- host: ticketing.dev` in `ingress-srv.yml` file, our domain will be different in prod and in dev. So we will have to move `ingress-srv.yml` to two different folders of `k8s-dev` and `k8s-prod`. Add `- ./infra/k8s-dev/*` to the manifests section of `skaffold.yml` file. Change the last line of the above `deploy-manifests.yml` to `- run: kubectl apply -f infra/k8s && kubectl apply -f infra/k8s-prod`.

- Do not forget to set the secrets on the production cluster (make sure the context is DO):

```
kubectl create secret generic jwt-secret --from-literal=jwt=thisIsASecret
kubectl create secret generic stripe-secret --from-literal=STRIPE_KEY=sk_test_OKBM67rLAdixSAhvuNohK2qw00GCjdgNGm
```

- Also do not forget to install ingress-nginx in the cluster (see the docs for install for DO):

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.34.1/deploy/static/provider/do/deploy.yaml
```

- By doing the above, a load balancer will be provided. You can see it by going DO dashboard -> Networking -> Load balancer. Because there is no host name in our ingress-nginx service, if you go to the ip of this load-balancer, we will get an nginx 404 error. So we have to buy a domain name and point it to our load balancer. You can buy a domain from `namecheap.com` for \$1.4: `john-ticketing-app.xyz`.
- In namecheap dashboard, change the nameserver to custom:

![](/md/76.jpg)

- Add the following:

![](/md/77.jpg)

- Go to DO -> Networking -> Domains -> Add `john-ticketing-app.xyz` domain

![](/md/78.jpg)

![](/md/79.jpg)

- Change the `ticketing.dev` in `infra/k8s-prod/ingress-srv.yml` to `www.john-ticketing-app.xyz`.
- There is currently a bug with ingress-nginx on Digital Ocean. You can read more about this bug here: https://github.com/digitalocean/digitalocean-cloud-controller-manager/blob/master/docs/controllers/services/examples/README.md#accessing-pods-over-a-managed-load-balancer-from-inside-the-cluster. To fix it, add the following to the bottom of your ingress-srv.yaml file.

```yml
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: "true"
    service.beta.kubernetes.io/do-loadbalancer-hostname: "www.john-ticketing-app.xyz"
  labels:
    helm.sh/chart: ingress-nginx-2.0.3
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.32.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
```

- You may recall that we configured all of our services to only use cookies when the user is on an HTTPS connection. This will cause auth to fail while we do this initial deploy of our app, since we don't have HTTPS setup right now. To disable the HTTPS checking, go to the `app.ts` file in the auth, orders, tickets, and payments services. At the cookie-session middleware, change `secure: process.env.NODE_ENV !== 'test',` to `secure: false,`.
- All of our services are in dev mode. To build them, create a separate dockerfile which has an extra line to build the app (something like `npm run build`) and change the image command to `node index`.

![](/md/80.jpg)
