# Basics

- Docker wants to make it easy to install and run software on any platform.
- Docker is a platform around creating and running containers: Docker client (Docker CLI), Docker server (Docker Daemon), Docker machine, Docker images, Docker hub, Docker compose.
- When you install Docker, Docker client (Docker CLI) and and Docker server (Docker Daemon) will be installed on your machine. Be careful to use `Linux containers`. After the installation, check `docker version` to check that you have docker.
- `Docker cli` is a tool that we are going to issue commands to.
- `Docker server` is a tool that creates images, upload/download images to/from docker hub, and maintain containers.
- `Docker image`: is a single file with all the deps and config required to run a program. So it has a `file system snapshot` + `startup command`. The file system snapshot has folders like `bin`, `home`, `usr`, `var`, `root`, `tmp`, `sys`, `dev`, `etc`, `proc`, ... like linux!
- `Docker container`: is an instance of an image. So it is _a running program_ (a process or set of processes) running in a `linux` environment with isolated set of hardware resources such as hard disk, memory, cpu, and network assigned to it.

## Running containers = creating + starting

### `docker run <image_name>`

- When you run `docker run <image_name>`,
  1. docker cli reaches out to docker server,
  2. docker server looks into its image cache to see if it has the program image,
  3. if it's the first time and it doesn't have the image, it connects to docker hub (repository of free public images) and downloads the program docker image,
  4. it creates a docker container out of it (it is just copying the file system from image),
  5. it runs the default program inside the container (start) and attaches the output of the container to the current terminal.
- Each time you execute `docker run ...` a new container will be built and run.
- If the program does something and then exits, the container will be shut down; but if it continues to run like a server, the container will remain. So a container has a _running_ program.
- Basically it is equivalent to:

```dos
docker create <image_name>

d1145084d39f204aadb69b377b569f5f280540c5e4f36f53dc7c3c8f909bbd0f
```

plus

```dos
docker start -a d1145084d39f204a
```

### `docker run <image_name> <command>`

- It overrides the default command. So when a new container is built, the command that we have specified will be run instead of the default command. Of course, the image should know how to run that command. Example (bash is present in busybox image file system):

```dos
docker run busybox echo hi there

hi there
```

```dos
docker run busybox ls

bin
dev
etc
home
proc
root
sys
tmp
usr
var
```

- Basically it is equivalent to:

```dos
docker create <image_name> <command>

14a167167cb81dfbd108fe9d4d045937ade243720e47e24bc7e217575bdec3ac
```

plus

```dos
docker start -a 14a167167cb81dfbd108fe9d4d0
```

## Listing containers

### `docker ps`

- list all running containers.
- To see some running containers, we need some program that runs for a while, like:

```dos
docker run busybox ping google.com
```

- In another terminal window:

```dos
docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d87d17abeed1        busybox             "ping google.com"   6 seconds ago       Up 4 seconds                            determined_lovelace
```

### `docker ps --all`

- list all containers (even those stopped).

## Restarting a stopped container

```dos
docker start -a <CONTAINER ID>
```

- It runs the startup command.
- Important: You cannot change the startup command of a container after its creation.

## Removing stopped containers

### `docker system prune`

## Getting logs from a container (running or stopped)

- If we want to see the output of a container that we don't access to its terminal (examples: we started a container without `-a`, or we started a container but then closed the terminal, or situations you will see later...).

### `docker logs <CONTAINER ID>`

## Stopping a container

### `docker stop <CONTAINER ID>`

- It send a `SIGTERM` for termination and gives 10 secs to the process to shut itself down, after that it will kill the process.

### `docker kill <CONTAINER ID>`

- It kills the process immediately.

## Executing another command in a running container

- For example, we run redis and we want to have access to redis-cli.

### `docker exec -it <CONTAINER ID> <command>`

- `-it` allows us to provide input to the container (`-t` is just for formatting the terminal in a nice and pretty way).

```dos
docker run redis
docker ps
docker exec -it 75286e1ee5fb redis-cli
```

- The most useful second command to run in a container is `sh` to open a shell (sh is like bash, powershell, and zsh) in the running container:

```dos
docker exec -it 75286e1ee5fb sh
cd /
ls
export b=5
echo $b
redis-cli
exit
exit
```

- You could run a container with shell as startup, but most of the time, you want to have your primary process (like a web server) running and then attach a shell to it; so not very common:

```dos
docker run -it busybox sh
```

# Create Docker Images

## Example

1. Create the `Dockerfile` (with no extension):

```dockerfile
# Use an existing docker image as a base
FROM alpine

# Download and install a dependency
RUN apk add --update redis

# Tell the image what to do when it starts as a container
CMD ["redis-server"]
```

2. Execute:

```dos
docker build -t johndoe/redis:latest .

Successfully built 5e3f8d79ef7f
Successfully tagged johndoe/redis:latest
```

3. Execute:

```dos
docker run 5e3f8d79ef7f
```

or

```dos
docker run johndoe/redis:latest
```

## Explanation

- Writing a Dockerfile is similar to tell a computer with no OS to install something.
- In a Dockerfile, we:
  1. Specify a base image.
  2. Run some commands to install additional programs.
  3. Specify the startup command.
- The Dockerfile will be executed by `docker server`.
- Each command in the Dockerfile has two parts: instruction (a single word) + argument.

### Build process

- The `.` specifies the directory of files/folders to use for the build.
- For each instruction except `FROM`,
  1. an intermediate container will be built out of the previous image,
  2. that container file system be modified by the current instruction (e.g, an application is downloaded and installed on it),
  3. a new image will be generated out of the current container (a file system snapshot),
  4. the intermediate container will be removed,
  5. the generated image will be fed to the next instruction.
- The final image will be the output of the entire process.
- If you run `docker build .` again, the output image of each instruction has been cached and will be used.
- If you change an instruction, the output image of each instruction until that instruction will be used from cache but after that line on down, new images will be generated and cached. So it is better to put the changes as far down as possible.
- The convention for tagging is `your_docker_id/repo_or_project_name:project_version`. The tag is actually the `project_version`. If you don't specify the tag, `latest` will be used.

### Common instructions

- Each instruction produces an image.

#### FROM

- We specify the base image that includes a set of programs that suits our needs.
- We can specify the tag of the image in front of a `:`.
- Many images offer an `alpine` version which is as compact as possible (e.g, `FROM node:alpine`).
- When the `docker server` executes this line, it will look at its cache and if it can't find the image, it will download it from `docker hub`.

#### RUN

- What comes after RUN has nothing to do with docker. It is just the startup command to be run on the intermediate container built out of the image extracted from previous step. In the example, `apk add --update redis` was `Alpine Linux package manager` command to install redis. If we had used `node` instead of `alpine`, we would have been used `npm` or `yarn`.

#### CMD

- It sets the default command of the image.
- The command is inside a bracket in case of the command having spaces in it (e.g CMD ["npm", "start"]).

#### COPY

- The first path points to the folder to copy from on your machine (relative to build context).
- The second path points to inside the container.

#### WORKDIR

- It changes the working directory from the root to what we specify in the docker container. Because in the root of the container, we have folders like `bin`, `lib`, ... and we don't want them to be replaced by copying files from the current machine to the container.

#### `docker run -p <port_on_local_host>:<port_inside_container> <image_name>`

- It is not an instruction in the Dockerfile, it is a runtime configuration.
- You can map the traffic to a port on your machine to be mapped to a port in the container.
- Note that the above is for incoming traffic and the docker container can freely reach out to the internet for outcoming traffic.

## Dockerizing a node project

- The whole purpose is to NOT INSTALL ANYTHING: node, npm , dependencies, ...

### Flow

1. Create the app

- Create `package.json` file:

```json
{
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "*"
  }
}
```

- Create `index.js`:

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("How are you doing");
});

app.listen(8080, () => {
  console.log("Listening on port 8080");
});
```

2. Create the Dockerfile

```dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY ./ ./
RUN npm install

CMD ["npm", "start"]
```

3. Build image from Dockerfile

```dos
docker build -t johndoe/simpleweb .
```

4. Run image as a container

```dos
docker run -p 5000:8080 johndoe/simpleweb
```

5. Use the app by navigating to `http://localhost:5000/`

   ![](/md/workdir.jpg)

The **problems** with the above project are

- For every change in the project, we have to build another image again and run it; so we have to use volumes (to be discussed later).
- Because the line `COPY ./ ./` has been invalidated due to changes in the project, it doesn't use cache for `RUN npm install` which can be very time-consuming. This problem can be attacked by separating the copies:

```dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY ./package.json ./
RUN npm install
COPY ./ ./

CMD ["npm", "start"]
```

## Dockerizing a project with multiple docker containers (node + redis)

- Each part of the project should have its own docker container.
- We are going to use `docker-compose` to simplify writing docker commands with many options and switches to configure and initialize docker containers (configurations such as building images, and network mapping).
- docker-compose is used to startup multiple docker containers at the same time and automatically connect them together; meaning that when two services are defined in a docker-compose file, they are automatically connected together and we don't have to map ports between them. We just have to map ports from our local machine.
- We create a `docker-compose.yml` file and put all those build commands and run commands with many options into a special syntax.
- Then, we run the following `to build and run` those containers defined in the yml file:

```dos
docker-compose up --build
```

- Or, we run the following just `to run` those containers:

```dos
docker-compose up
```

- To launch in the background:

```dos
docker-compose up -d
```

- To stop containers:

```dos
docker-compose down
```

- To see the status of containers in a docker-compose (unlike `docker ps`, this should be run from the same directory as the one that the docker-compose file resides in):

```dos
docker-compose ps
```

![](/md/compose.jpg)

### Flow

1. Create the app:

- Create `package.json` file.
- The redis in this file is to build a redisClient in the express app:

```json
{
  "dependencies": {
    "express": "*",
    "redis": "2.8.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

- Create `index.js`:

```js
const process = require("process");
const express = require("express");
const redis = require("redis");
const { promisify } = require("util");

const app = express();

const redisClient = redis.createClient({
  host: "redis-server", //It is corresponding to the service name defined in docker-compose.yml file.
  port: 6379,
});
redisClient.get = promisify(redisClient.get);
redisClient.set("visits", 0);

app.get("/", async (req, res) => {
  const visits = await redisClient.get("visits");

  res.send(`Number of visits is ${visits}`);

  redisClient.set("visits", parseInt(visits) + 1);
});

app.get("/exit", async (req, res) => {
  process.exit(0); //exit without error. It will stop the container.
});

app.get("/error", async (req, res) => {
  process.exit(1); //exit with error. It will stop the container too.
});

app.listen(8081, () => {
  console.log("Listening on port 8081");
});
```

2. Create the Dockerfile in the root of the project to create node app image (the redis image will be directly downloaded from docker hub):

```dockerfile
FROM node:alpine

WORKDIR /app

COPY package.json .
RUN npm install
COPY . .

CMD ["npm", "start"]
```

3. Create the docker-compose.yml in the root of the project:

```yml
# Version of docker-compose
version: "3"

# We are defining the type of the containers that we have in our project in the services section.
services:
  redis-server: # WE called it redis-server.
    # We could specify a restart policy for redis-server too.
    image: "redis" # It means that, use this image to build a container.

  node-app:
    restart: always # We have 4 restart policies:
    # "no" (it has to be inside "")
    # always (for web servers, use this one)
    # on-failure (for worker containers, use this one)
    # unless-stopped (it means always restart unless we run docker kill or stop ...)
    build: . # In the current directory, look for a Dockerfile.
    ports:
      - "5000:8081" # - means array.
```

4. Run

```dos
docker-compose up -d
```

- Note that, if we change the project, we should use `--build` flag.

5. Use the app by navigating to `http://localhost:5000/`

# Development Workflow

- We want to establish a development workflow which should be good for `development`, `testing`, and `deployment`.
- We will create a github repo with two branches: `feature` and `master`.
- We pull from and push to feature branch and then make a `pull request` to merge with the master.
- After merging, a series of events should happen:
  1. A `CI provider` should pull the code and run some tests.
  2. Upon successful test results, the CI provider pushes the code to a `hosting environment` like DigitalOcean or AWS.
- We can establish such workflow without docker; but `with docker` it is a lot easier.

## Single container project (react)

- Install the app by:

```dos
npx create-react-app frontend
```

- In a react app, there are three commands:
- Starts a development server:

```dos
npm run start
```

- Runs tests associated with the project:

```dos
npm run test
```

- Builds a production version of the app:

```dos
npm run build
```

- So we want two Dockerfiles: `Dockerfile.dev` to start the app with `npm run start` and `Dockerfile` to start the app with `npm run build`.

### Development

- We don't want to rebuild the image and create a new container, whenever we change the code. We want the docker container to have access to our local machine for the development code base.
- But we do want that the dependencies are installed in the container (this is the whole point of using docker!). So delete the `node_modules` from the local machine.

- Create `Dockerfile.dev` in the root of the project:

```dockerfile
FROM node:alpine

WORKDIR /app

COPY package.json .
RUN npm install
# The line below is not necessary in this file for development but we keep it as a reminder for the production.
COPY . .

CMD ["npm", "run", "start"]
```

- To specify the file to build the image, use `-f Dockerfile.dev`:

```dos
docker build -f Dockerfile.dev .

Successfully built b0ec8aeb70b1
```

- Run the image by:

```dos
docker run -it -p 3000:3000 -v /app/node_modules -v ${pwd}:/app b0ec8aeb70b1
```

- The first `-v /app/node_modules` is like a placeholder for not to map that folder to the `present working directory`.
- In MacOS, you should use `$(pwd)` instead of `${pwd}`.
- React needs an interactive terminal, so `-it` is needed.

#### Use of docker-compose

- Because the commands are lengthy, although we have just one container, but in order to simplify the commands, we create a `docker-compose` file:

```yml
version: "3"
services:
  react-app:
    build: # Because here we want to build from Dockerfile.dev and not Dockerfile, we can't use `.`.
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
    stdin_open: true # React needs an interactive terminal, so `-it` is needed.
```

- Now, we can just use:

```dos
docker-compose up
```

### Testing

- For executing `tests`, install these packages:

```dos
npm i @testing-library/jest-dom @testing-library/react
```

and then delete the `node_modules` from local machine.

Then, rebuild the docker-compose by (remember, you build whenever you want to change the snapshot; like installing dependencies):

```dos
docker-compose up --build
```

#### Solution 1

- Attach the test process to the existing running container by:

```dos
docker ps
docker exec -it 9b59c1eff651 npm run test
```

#### Solution 2

- Change the `docker-compose.yml` to be:

```yml
version: "3"
services:
  react-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
    stdin_open: true
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    stdin_open: true
    command: ["npm", "run", "test"] # With this, we override the command in the Dockerfile.dev.
```

- The problem with this approach is that, because we are running two containers with one command, we don't have access to `stdin` of neither of those.

### Production

- In production we don't have access to development server. So we need `nginx` server to serve the built files.
- Note that, first we only need the `build` folder and not any `node_modules` (we need node_modules only to run `npm run build`) and second, we should install and configure nginx in the image. Thus, we are dealing with **`multi-step builds`** (we need both `node` and `nginx` base images). So create `Dockerfile`:

```dockerfile
FROM node:alpine as builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
EXPOSE 80
# This EXPOSE instruction is purely for AWS Beanstalk. So it knows how to do the port mapping when running the container out of this docker image.
COPY --from=builder /app/build /usr/share/nginx/html
# We don't need any command because that nginx base image automatically runs the server
```

- Run:

```dos
docker build .

Successfully built 2687cccf92a0
```

- Then:

```dos
docker run -p 8080:80 2687cccf92a0
```

- Navigate to `http://localhost:8080/`.

### Github, Travis, and AWS

- Add a github repo (`project1`) to the project and push the project to it.
- Travis is used to test and deploy.
- Login to `Travis-ci.org` using github.
- Go to the repositories section and turn the switch on for the project. So Github will notify Travis, when there is a change.
- Create a `.travis.yml` file in the root of the project:

```yml
sudo: required
services:
  - docker

before_install:
  - docker build -t johndoe/project1 -f Dockerfile.dev .

script:
  - docker run -e CI=true johndoe/project1 npm run test ## -e CI=true is for setting env variable for telling jest to exit with status code after running tests
```

- `Elastic Beanstalk` is great for running one container at a time. Because it has a `Load Balancer` which talks to an `EC2` instance (our docker container will run on this EC2), when the load is increased, that EC2 instance will be replicated and the load balancer will route the request to the node with the least traffic.
- In the `EBS` dashboard, click on the `Create New Application` on the top-right.
- Give `Application Name` and click on `Create`.
- Click on `Create one now` to create an environment for the application.
- Choose `Web server environment` and click `Select`.
- Scroll-down to `Base configuration`, under `Platform`, select `Docker`.
- Click on `Create environment`.
- Add some scripts to the `.travis.yml` file for deploying:

```yml
sudo: required
services:
  - docker

before_install:
  - docker build -t johndoe/project1 -f Dockerfile.dev .

script:
  - docker run -e CI=true johndoe/project1 npm run test

deploy:
  provider: elasticbeanstalk
  region: "us-west-2" # It is the region that your EBS instance is created in.
  app: "docker-react" # The name of the app in the EBS dashboard.
  env: "Docker-react-env" # The name of the environment which is created (It can be found in the EBS dashboard: All Applications > docker-react > Docker-react-env)
  bucket_name: "elasticbeanstalk-us-west-2-306476627547" # The name of the s3 bucket that Travis will put the .zip file of the project into. This s3 bucket is automatically generated when creating an EBS instance. To find it, search for s3 in the services. In the list, look for something elasticbeanstalk-us-west-2... This bucket is used for all the EBS environments (it is not just for this project). Every project is a folder in this s3 bucket.
  bucket_path: "docker-react" # By default it is the same as the app name.
  on:
    branch: master # Whenever master is updated, try to push to AWS.
```

- Go to `Identity and Access Management` section.
- We are going to generate a new user to give it to travis. So click on `Users` in the left panel.
- Click `Add user` at the top and provide a user name such as: docker-react-travis-ci.
- Choose the `Access Type` as `Programmatic access` only.
- Hit `Next: Permissions` and select `Attach existing policies directly` tab.
- Search for `beanstalk` and select the one with the description of `Provide full access to AWS Elastic Beanstalk`.
- Hit `Next: Review` and `Create user`.
- Copy the `Access key ID` and `Secret access key`.
- Select the project in travis, click on `More options` then on `Setting` and add the values above to the `Environment Variables`. For names use something like AWS_ACCESS_KEY and AWS_SECRET_KEY.
- Add some scripts to the `.travis.yml` file for giving access to AWS:

```yml
sudo: required
services:
  - docker

before_install:
  - docker build -t johndoe/project1 -f Dockerfile.dev .

script:
  - docker run -e CI=true johndoe/project1 npm run test

deploy:
  provider: elasticbeanstalk
  region: "us-west-2"
  app: "docker-react"
  env: "Docker-react-env"
  bucket_name: "elasticbeanstalk-us-west-2-306476627547"
  bucket_path: "docker-react"
  on:
    branch: master
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key:
    secure: "$AWS_SECRET_KEY"
```

## Multiple containers project

### Development

![](/md/dev.jpg)

#### Flow

1. Create a folder for the `worker` app:

- Create `package.json` file.

```json
{
  "dependencies": {
    "config": "^3.3.1",
    "redis": "2.8.0"
  },
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "devDependencies": {
    "nodemon": "^1.18.3"
  }
}
```

- Create `index.js`:

```js
const config = require("config");
const redis = require("redis");

const redisClient = redis.createClient({
  host: config.get("redisHost"),
  port: config.get("redisPort"),
  retry_strategy: () => 1000,
});
const redisSubscriber = redisClient.duplicate();

function fib(index) {
  if (index < 2) return 1;
  return fib(index - 1) + fib(index - 2);
}

redisSubscriber.on("message", (channel, message) => {
  redisClient.hset("values", message, fib(parseInt(message)));
});
redisSubscriber.subscribe("insert");
```

- Create a `config` folder with the file `custom-environment-variables.json` in it:

```json
{
  "redisHost": "REDIS_HOST",
  "redisPort": "REDIS_PORT"
}
```

2. Create a folder for the `server` app:

- Create `package.json` file.

```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "config": "^3.3.1",
    "cors": "^2.8.5",
    "express": "^4.17.1",
    "pg": "^7.14.0",
    "redis": "2.8.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.3"
  }
}
```

- Create `index.js`:

```js
const config = require("config");
const express = require("express");
const cors = require("cors");
const { promisify } = require("util");

const app = express();
app.use(cors());
app.use(express.json());

// Postgres Client Setup
const { Pool } = require("pg");
const pgClient = new Pool({
  user: config.get("pgUser"),
  password: config.get("pgPassword"),
  database: config.get("pgDatabase"),
  host: config.get("pgHost"),
  port: config.get("pgPort"),
});
pgClient.on("error", () => console.log("Lost Postgres connection"));

pgClient
  .query(
    `
    CREATE TABLE IF NOT EXISTS values (
      id serial PRIMARY KEY,
      number integer NOT NUll
    )
`
  )
  .catch((err) => console.log(err));

// Redis Client Setup
const redis = require("redis");
const redisClient = redis.createClient({
  host: config.get("redisHost"),
  port: config.get("redisPort"),
  retry_strategy: () => 1000,
});
redisClient.hgetall = promisify(redisClient.hgetall);
const redisPublisher = redisClient.duplicate();

// Express route handlers
app.get("/", (req, res) => {
  res.send("Hi there");
});

app.get("/values/all", async (req, res) => {
  const values = await pgClient.query("SELECT * from values");

  res.send(values.rows);
});

app.get("/values/current", async (req, res) => {
  const values = await redisClient.hgetall("values");

  res.send(values);
});

app.post("/values", async (req, res) => {
  const index = req.body.index;

  if (parseInt(index) > 40) {
    return res.status(422).send("Index too high");
  }

  redisClient.hset("values", index, "Nothing yet!");
  redisPublisher.publish("insert", index);
  pgClient.query("INSERT INTO values(number) VALUES($1)", [index]);

  res.send({ working: true });
});

app.listen(5000, (err) => {
  console.log("Listening");
});
```

- Create a `config` folder with the file `custom-environment-variables.json` in it:

```json
{
  "pgUser": "PGUSER",
  "pgPassword": "PGPASSWORD",
  "pgDatabase": "PGDATABASE",
  "pgHost": "PGHOST",
  "pgPort": "PGPORT",
  "redisHost": "REDIS_HOST",
  "redisPort": "REDIS_PORT"
}
```

3. Create a folder for the `client` app:

- Create `package.json` file.

```json
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.4.2",
    "react-dom": "^16.4.2",
    "react-scripts": "1.1.4",
    "react-router-dom": "4.3.1",
    "axios": "0.18.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "devDependencies": {
    "@testing-library/jest-dom": "^5.5.0",
    "@testing-library/react": "^10.0.3"
  }
}
```

- Create `Fib.js` component:

```js
import React, { useState, useEffect } from "react";
import axios from "axios";

const Fib = () => {
  const [seenIndexes, setSeenIndexes] = useState([]);
  const [values, setValues] = useState({});
  const [index, setIndex] = useState("");

  useEffect(() => {
    fetchValues();
    fetchIndexes();
  }, [fetchValues, fetchIndexes]);

  const fetchValues = async () => {
    const { data: values } = await axios.get("/api/values/current");

    setValues(values);
  };

  const fetchIndexes = async () => {
    const { data: seenIndexes } = await axios.get("/api/values/all");

    setSeenIndexes(seenIndexes);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    await axios.post("/api/values", { index });

    setIndex("");
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <label>Enter your index:</label>
        <input value={index} onChange={(e) => setIndex(e.target.value)} />
        <button>Submit</button>
      </form>

      <h3>Indexes I have seen:</h3>
      {seenIndexes.map(({ number }) => number).join(", ")}

      <h3>Calculated Values:</h3>
      {Object.keys(values).map((key) => {
        return (
          <div key={key}>
            For index {key} I calculated {values[key]}
          </div>
        );
      })}
    </div>
  );
};

export default Fib;
```

- Create `OtherPage.js` component:

```js
import React from "react";
import { Link } from "react-router-dom";

const OtherPage = () => {
  return (
    <div>
      Im some other page
      <Link to="/">Go back to home page!</Link>
    </div>
  );
};

export default OtherPage;
```

- Create `App.js` component:

```js
import React from "react";
import { BrowserRouter as Router, Route } from "react-router-dom";
import OtherPage from "./OtherPage";
import Fib from "./Fib";

const App = () => {
  return (
    <Router>
      <div>
        <Route exact path="/" component={Fib} />
        <Route path="/otherpage" component={OtherPage} />
      </div>
    </Router>
  );
};

export default App;
```

4. Create a folder for the `nginx` app with a `default.conf` file:

```conf
upstream client { # We call it client here.
    server client:3000; # client is actually a url (service) in the docker-compose file.
}

upstream api { # We call it api here.
    server api:5000; # api is actually a url (service) in the docker-compose file.
}

server {
    listen 80; # This 80 is not important because we will change it in docker-compose.

    location / { # Route all requests to home to the client upstream. (Note that it will redirect /otherpage correctly.)
        proxy_pass http://client;
    }

    location /sockjs-node { # Route all requests to /sockjs-node to the client upstream. This web socket connection is used by React app.
        proxy_pass http://client; # to the client upstream.
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    location /api { # Route all requests to /api to the api upstream.
        rewrite /api/(.*) /$1 break; # Chop off the `/api` section. It is for convenience. As you can see, we didn't use `/api` in our routes in the backend. (.*) is regex and $1 is the match of that regex. break means do not apply any other rewrite rules.
        proxy_pass http://api; # to the api upstream.
    }
}
```

5. Create the `Dockerfile.dev` in the root of `each project` to create image:

5.1. `worker`, `server`, and `client` projects:

```dockerfile
# For `server` use `FROM node:12.13.0-alpine AS alpine` instead! PG won't connect otherwise!
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
# For `client` use `CMD ["npm", "run", "start"]` instead.
CMD ["npm", "run", "dev"]
```

5.2. `nginx` project:

```dockerfile
FROM nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf
# So we just need to configure default.conf for nginx and copy it to the container to build out own image.
```

6. Create the `docker-compose.yml` in the root of the project:

```yml
version: "3"
services:
  postgres:
    image: postgres:latest
    environment:
      - POSTGRES_PASSWORD=postgres
    container_name: fib_postgres
  adminer: # It is a GUI to connect to postgres
    image: adminer
    ports:
      - 8080:8080
    container_name: fib_adminer
  redis:
    image: "redis:latest"
    container_name: fib_redis
  nginx:
    restart: always
    build:
      context: ./nginx
      dockerfile: Dockerfile.dev
    container_name: fib_nginx
    ports:
      - "3050:80"
  api:
    depends_on:
      - postgres
    build:
      context: ./server
      dockerfile: Dockerfile.dev
    container_name: fib_api
    volumes:
      - /app/node_modules
      - ./server:/app
    environment:
      - PGUSER=postgres # It is set at the runtime (not build time).
      - PGPASSWORD=postgres # If we don't specify '=postgres', the value will be taken from local computer.
      - PGDATABASE=postgres
      - PGHOST=postgres
      - PGPORT=5432
      - REDIS_HOST=redis
      - REDIS_PORT=6379
  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - ./client:/app
    container_name: fib_client
  worker:
    build:
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker:/app
    container_name: fib_worker
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
```

7. Run

```dos
docker-compose up
```

8. Use the app by navigating to `http://localhost:3050/`

### Github, Travis, and AWS

- We want:

  1. Push code to github.
  2. Travis automatically pulls repo.
  3. Travis builds a test image and test the code.
  4. Travis builds a production image.
  5. Travis pushes built production image to Docker hub.
  6. Travis pushes pushes project to AWS EBS.
  7. AWS EBS pulls images from Docker hub and deploys.

![](/md/prod.jpg)

1. Create the `Dockerfile` in the root of `each project` to create image for production:

**For `worker`**:

```dockerfile
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

**For `server`**:

```dockerfile
FROM node:12.13.0-alpine AS alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "start"]
```

**For `nginx`**:

- We could use another default.conf for production but why should we?! If we used another default.conf, it would be better to remove websocket section too. Because we wouldn't need those too.

```dockerfile
FROM nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf

```

**For `client`**:

- First create a `default.conf` in an `nginx` folder inside the `client` project.
- This is an nginx server which is a substitute to React development server. We have to expose 3000 because in the **main** `default.conf`, we redirected those to 3000. So we need to modify this instance default.conf.

```conf
server {
  listen 3000;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html; # This line would get the Nginx server to work correctly when using React Router.
  }
}
```

- Then create the `Dockerfile` in the `client` project:

```dockerfile
FROM node:alpine as builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/build /usr/share/nginx/html
```

2. Add a github repo (`complex`) to the project and push the project to it.
3. Login to `Travis-ci.org` using github, go to the repositories section, and turn the switch on for the project. So Github will notify Travis, when there is a change.
4. Create a `.travis.yml` file in the root of the project:

```yml
sudo: required

services:
  - docker

before_install:
  - docker build -t johndoe/complex-react-test -f ./client/Dockerfile.dev ./client

script:
  - docker run -e CI=true johndoe/complex-react-test npm run test

after_success:
  - docker build -t johndoe/complex-nginx ./nginx
  - docker build -t johndoe/complex-client ./client
  - docker build -t johndoe/complex-server ./server
  - docker build -t johndoe/complex-worker ./worker
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_ID" --password-stdin # Login to the docker CLI as me
  - docker push johndoe/complex-nginx # docker push, pushes an image to the docker hub.
  - docker push johndoe/complex-client
  - docker push johndoe/complex-server
  - docker push johndoe/complex-worker
```

5. Select the project in travis, click on `More options` then on `Setting` and add the values for your `DOCKER_PASSWORD` and `DOCKER_ID` to the `Environment Variables`. This way, after travis is finished, the build files will be uploaded to `hub.docker.com`.

6. Deploying to AWS:

- When we had only one dockerfile, EBS knew how to build a container, but now, there are multiple images in different folders. So we create `Dockerrun.aws.json` in the root of the project (which is kinda like `docker-compose.yml` file with the differences: (1) our images are already produced and (2) instead of services, we have container definitions) to tell EBS where to pull our images, what resources allocate to each one, how to do the port mappings, ...
- In fact, EBS doesn't know how to run containers and behind the scene (`Amazon Elastic Container Service`, ECS), use these container definitions.

```json
{
  "AWSEBDockerrunVersion": 2,
  "containerDefinitions": [
    {
      "name": "client", // We give it this name. Just be consistent.
      "image": "johndoe/complex-client",
      "hostname": "client", // This is the name that can be accessed by other containers, so it is like the name of the service in docker-compose.yml file. We used that file in development and specified our env variables based on the service names in it. Also, we used those service names in the nginx default.conf files, so we have to choose that name.
      "essential": false, //If true, If this container crashes, all other containers need to be shut down. At least, one container needs to be marked as essential.
      "memory": 128 //This is required for each container (in MB).
    },
    {
      "name": "server",
      "image": "johndoe/complex-server",
      "hostname": "api",
      "essential": false,
      "memory": 128
    },
    {
      "name": "worker",
      "image": "johndoe/complex-worker",
      "hostname": "worker", //It is not required here. Because nothing reaches out worker.
      "essential": false,
      "memory": 128
    },
    {
      "name": "nginx",
      "image": "johndoe/complex-nginx",
      "hostname": "nginx", //It is not required too. Because nothing reaches out nginx.
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80, //The port of EBS machine. There is just one machine running all these containers :(
          "containerPort": 80
        }
      ],
      "links": ["client", "server"] //In docker-compose world, the link between containers was automatic. Here, we need to specify by the container name that we gave. It is uni-directional (we need to refer client and server in the nginx container and not vice versa).
    }
  ]
}
```

- `Elastic Beanstalk` is not ideal for running a multiple container project. Because all the containers are running in one machine (the four containers we defined above). It is better to use `Kubernetes`. With `EBS` the whole project (only the four containers defined above, not redis and postgres) will be replicated! But we use EBS anyway.

  - Go to EBS service in the aws.
  - In the `EBS` dashboard, click on the `Create New Application` on the top-right.
  - Give `Application Name` and click on `Create`.
  - Click on `Create one now` to create an environment for the application.
  - Choose `Web server environment` and click `Select`.
  - Scroll-down to `Base configuration`, under `Platform`, select `Multi-container Docker`.
  - Click on `Create environment`.

- Instead of defining redis and postgres containers (sometimes we have to define them in containers), we are going to use `AWS ElastiCache` and `AWS Relational Database Service (RDS)`. Because, (1) these services are tailored by professionals, (2) super easy to scale, (3) built-in logging and maintenance, (4) better security, (5) easier to migrate off of EBS, and (6) automated backups and rollbacks.
- We have three different instances (EBS, ElastiCache, and RDS), we want to make them talk to each other:
  - When we created an EBS instance in a region, a default `Virtual Private Cloud (VPC)` network has been created for us. So for every region, we will get a different default VPC.
  - Each instance type in a VPC has a `security group` which is basically a set of firewall rules.
  - For example in an EBS instance, port 80 is open to the outside world (for incoming traffic) and there is no limitation for outcoming traffic in the default security group attached to it.
  - So we will define a security group to allow any traffic from any other AWS services that has this security group and we will attach it to our EBS, ElastiCache, and RDS instances.
- `Creating` and `connecting` ElastiCache and RDS services:

  - Make sure that you are in the same region as the EBS instance, then search for **RDS** service.
  - Scroll-down and click on `Create database`.
  - Select PostgreSQL and click on `Next`.
  - Select `DB instance class` and `Allocated storage`.
    - Give a name to this RDS instance (this is not database name in postgres),
    - and give a master username and password
    - and then click on `Next`.
  - Select the default VPC of this region to put this RDS instance to,
    - select the `Public accessibility` to `No`,
    - give a database name,
    - select backup retention period,
    - and then click on `Create database`.
  - Make sure that you are in the same region as the EBS instance, then search for **ElastiCache** service.
  - Click on the Redis in the left panel and then click on `Create`.
  - In the `Create your Amazon ElastiCache cluster` page:
    - give a name to this Redis instance,
    - select the `Node type` (it can be expensive),
    - select None for `Number of replicas`,
    - give a name for `Subnet group` (it has something to do with security),
    - select the default VPC of this region to put this ElastiCache instance to,
    - Check that two subnets to give access to those,
    - scroll-down and click on `Create`.
  - Go to VPC service. On the left panel, under `Security` section, click on **Security Groups**, and then click on `Create Security Group`, on the pop-up:
    - give a name to this security group,
    - select the default VPC on the dropdown,
    - click on `Yes, Create`.
  - Select the newly-created security group and go to `Inbound rules`:
    - on the `Source` column, select the current security group to allow traffic from this security group.
    - click on `Save`.
  - For **attaching security** group, go to ElastiCache service,
    - click on the `Redis` on the left panel,
    - select the checkbox for our instance,
    - click on `Modify` tab,
    - on the pop-up window, click on the `pencil` near the VPC Security Groups(s),
    - check the security group that we just created, click on `Save`, and then on `Modify`.
  - Go to RDS service,
    - click on the `Instances` on the left panel,
    - click on the instance,
    - scroll-down to `Details` section and click on `Modify`,
    - scroll-down to `Network & Security` section and add that security group from the dropdown,
    - scroll-down and click on `Continue`,
    - change to `Apply Immediately` on `Scheduling of Modifications` and click on `Modify DB Instance`.
  - Go to EBS service and click on our application,
    - click on the `Configuration` on the left panel,
    - click on `Modify` under `Instances` card,
    - scroll-down to `EC2 security groups` section and check the security group that we just created,
    - click on `Apply`, and then on `Confirm`.
  - To tell the different containers in our EBS instance on how to connect to RDS and ElastiCache, we have to **set the environment variables**. For setting environment variables for our EBS instance, go to EBS service and click on our application,
    - click on the `Configuration` on the left panel,
    - click on `Modify` under `Software` card,
    - scroll-down and give the environment variables that we used in docker-compose file.
    - For REDIS_HOST and PGHOST, we have to give the url of instances that we have in AWS. So go to our redis instance and copy the `Primary Endpoint` of the redis instance (without the :6379 at the end) and go to our postgres instance and copy `Endpoint` for the postgres instance.
    - For PGUSER, PGPASSWORD, and PGDATABASE, use the ones that used in creation of postgres instance.
    - Click on `Apply`.
    - Note that here all the containers have access to this EBS environment variables.
  - **Creating a user for travis**:
    - Go to `Identity and Access Management` section.
    - We are going to generate a new user to give it to travis. So click on `Users` in the left panel.
    - Click `Add user` at the top and provide a user name such as: complex-travis-ci.
    - Choose the `Access Type` as `Programmatic access` only.
    - Hit `Next: Permissions` and select `Attach existing policies directly` tab.
    - Search for `beanstalk` and select the one with the description of `Provide full access to AWS Elastic Beanstalk`.
    - Hit `Next: Review` and `Create user`.
    - Copy the `Access key ID` and `Secret access key`.
    - Select the project in travis, click on `More options` then on `Setting` and add the values above to the `Environment Variables`. For names use something like AWS_ACCESS_KEY and AWS_SECRET_KEY.

- Append the following script to the `.travis.yml` file for deploying and giving access to AWS (In reality travis will copy all the files to the bucket but what aws is really care about is only the `Dockerrun.aws.json`. Because we already created the images and uploaded to docker hub):

```yml
deploy:
  provider: elasticbeanstalk
  region: "us-west-2" # It is the region that your EBS instance is created in.
  app: "complex" # The name of the app in the EBS dashboard.
  env: "complex-env" # The name of the environment which is created (It can be found in the EBS dashboard: All Applications > docker-react > Docker-react-env)
  bucket_name: "elasticbeanstalk-us-west-2-306476627547" # The name of the s3 bucket that Travis will put the .zip file of the project into. This s3 bucket is automatically generated when creating an EBS instance. To find it, search for s3 in the services. In the list, look for something elasticbeanstalk-us-west-2... This bucket is used for all the EBS environments (it is not just for this project). Every project is a folder in this s3 bucket.
  bucket_path: "complex" # By default it is the same as the app name.
  on:
    branch: master # Whenever master is updated, try to push to AWS.
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key:
    secure: "$AWS_SECRET_KEY"
```

- Commit to the master branch and see the magic works!
- If there is an error, click on the `Logs` in the left panel of the EBS instance and then click on `Request Logs` and choose `Last 100 Lines`.
- Navigate to the link of the app in the EBS instance.
