# Setup

- Node is a runtime environment to execute JS code.
- To see if you have node installed on our machine:

```dos
node --version
or
node -v
```

- To run a node program:

```dos
node app.js
```

# Node Module System

- Node is modular. Each file is a module and we can define variables, functions, classes (which are actually constructor functions) in each file. In `C#`, every file is a class or interface.

## Global object

These are members of `global` object (`global.console.log("hi");`):

```js
console.log("Hi, there!");

myVar = setTimeout(() => {
  console.log("Hello");
}, 3000);
clearTimeout(myVar);

myVar = setInterval(() => {
  console.log("Hello");
}, 3000);
clearInterval(myVar);
```

## Module object

- In each file, we have access to `module`, `exports`, `require`, `__dirname`, and `__filename` objects which are scoped to that file/module. It is not global.
- Every `js` file is a module and all variables and functions defined in that file are scoped to that module. If we want to make them available, we have to export them. It is like making members (fields, props, methods) public in c#.
- Suppose, we have a file `logger.js`:

```js
module.exports = {
  log, //a function
  url, //a variable
};
//or
module.exports = log;
```

- To import the module from another file:

```js
const { log } = require("./logger"); // named export
//or
const log = require("./logger"); // default export -> you can rename it on the fly
```

- To see the module directory and file:

```js
console.log(__dirname); //C:\Users\Ben\Desktop\js
console.log(__filename); //C:\Users\Ben\Desktop\js\main.js
```

## Path Module

```js
const path = require("path");

var pathObj = path.parse(__filename);

console.log(pathObj);
/*
{
  root: 'C:\\',
  dir: 'C:\\Users\\Ben\\Desktop\\js',
  base: 'main.js',
  ext: '.js',
  name: 'main'
}
*/
```

## OS Module

```js
const os = require("os");

console.log(os.totalmem()); //8534650880
console.log(os.freemem()); //4315254784
```

## File System Module

- Methods in this module come in pairs (sync and async).
- Always use `async` ones; like the example below.
- It receives a callback as the second argument.

```js
const fs = require("fs");

fs.readdir("./", (err, files) => console.log(files)); //[ 'main.js' ]

// Read file
var http = require("http");
var fs = require("fs");
http
  .createServer(function (req, res) {
    fs.readFile("demofile1.html", function (err, data) {
      res.writeHead(200, { "Content-Type": "text/html" });
      res.write(data);
      return res.end();
    });
  })
  .listen(8080);

// Create or append file
var fs = require("fs");

fs.appendFile("mynewfile1.txt", "Hello content!", function (err) {
  // If the file does not exist, the file will be created. If exists, it will be appended.
  if (err) throw err;
  console.log("Saved!");
});

// Create or replace file
var fs = require("fs");

fs.writeFile("mynewfile3.txt", "Hello content!", function (err) {
  // If the file does not exist, a new file, containing the specified content, will be created. If exists, it will be replaced.
  if (err) throw err;
  console.log("Saved!");
});

// Delete file
var fs = require("fs");

fs.unlink("mynewfile2.txt", function (err) {
  if (err) throw err;
  console.log("File deleted!");
});

// Rename file
var fs = require("fs");

fs.rename("mynewfile1.txt", "myrenamedfile.txt", function (err) {
  if (err) throw err;
  console.log("File Renamed!");
});
```

## URL Module

```js
var url = require("url");
var adr = "http://localhost:8080/default.htm?year=2017&month=february";
var q = url.parse(adr, true);

console.log(q.host); //returns 'localhost:8080'
console.log(q.pathname); //returns '/default.htm'
console.log(q.search); //returns '?year=2017&month=february'

var qdata = q.query; //returns an object: { year: 2017, month: 'february' }
console.log(qdata.month); //returns 'february'
```

## Events Module

```js
const EventEmitter = require("events");
const emitter = new EventEmitter();

// Register a listener
emitter.on("messageLogged", (arg) => {
  console.log("Listener called", arg);
});

// Raise an event
emitter.emit("messageLogged", { id: 1, time: new Date() }); // Listener called { id: 1, time: 2020-04-23T04:57:20.865Z }
```

### Real world use

In the `logger.js`:

```js
const EventEmitter = require("events");

class Logger extends EventEmitter {
  log(msg) {
    console.log(msg);
    this.emit("messageLogged", { msg, time: new Date() });
  }
}

module.exports = Logger;
```

In the `main.js`:

```js
const Logger = require("./logger");
const logger = new Logger();

logger.on("messageLogged", (arg) => {
  console.log("Listener called", arg);
});

logger.log("Hi, there!");
//Hi, there!
//Listener called { msg: 'Hi, there!', time: 2020-04-23T05:05:00.469Z }
```

## Http Module

- Express is built on the `Http` module in node. We use that.

```js
const http = require("http");
const url = require("url");

http
  .createServer(function (req, res) {
    res.writeHead(200, { "Content-Type": "text/html" });
    console.log(req.url); // /?year=2020&month=7
    // Use the url module to turn the querystring into an object
    var q = url.parse(req.url, true).query; // url.parse(req.url, true) will be an object which has query, path, href, protocol, host, ... properties
    var txt = q.year + " " + q.month; // http://localhost:8080?year=2020&month=2
    res.end(txt);
  })
  .listen(8080);
```

# NPM

## Introduction

```dos
npm -v
```

- To initialize a node project (create a `package.json` file):

```dos
npm init -y
or
npm init --yes
```

## Installing a dependency

- To install a package globally:

```dos
npm i -g npm
```

- To install a dependency

After initializing a node project, you can install dependencies in the project.

```dos
npm i lodash
```

- To install a specific package:

```dos
npm i customPackage@6.13.4
```

## Use an installed package

```js
var _ = require("lodash");
```

Node will look into 1. Core modules, 2. File or folders, and 3. node_modules to resolve a require.

- We don't commit `node_modules` to the version control repository (add `node_modules` to the `.gitignore`), with:

```dos
npm i
```

all the dependencies will be restored.

## Versioning

Major.Minor.Patch
4.13.6

- Major means adding features with breaking changes.
- Minor means adding features without breaking changes.
- Patch means bug fixes.

Different types of dependencies in `package.json`:

- ^4.13.6 is equal to 4.x.x
- ~4.13.6 is equal to 4.13.x
- 4.13.6 is equal to 4.13.6

## devDependencies

```dos
npm i nodemon --save-dev
```

## Uninstall a dependency

```dos
npm un mongoose
```

# Console apps

```dos
npm i chalk yargs
```

- `Chalk` is for changing the console color:

```js
const chalk = require("chalk");

console.log(chalk.blue("Hello world!"));
console.log(chalk.red.inverse("Error!"));
```

- `Yargs` helps you build interactive command line tools, by parsing arguments and generating an elegant user interface.
- Create a file `temp.js`:

```js
const yargs = require("yargs");

// Customize yargs version
yargs.version("1.1.0");

yargs.command({
  command: "greet",
  describe: "Greets a person",
  builder: {
    name: {
      describe: "Person's name",
      demandOption: true,
      type: "string",
    },
  },
  handler(argv) {
    console.log(`Hello ${argv.name}!`);
  },
});

yargs.parse();
```

- For running yargs apps:

![](/md/yargs.jpg)

# Express

## Setup

```dos
mkdir project
cd project
npm init -y
npm i express config @hapi/joi
npm i -g nodemon
```

### root folder

- In the root of the project, create `app.js`:

```js
const express = require("express");
const app = express();

require("./startup/routes")(app);

module.exports = app;
```

- Next to the file above, create `index.js`:

```js
const app = require("./app");
const config = require("config");

const PORT = process.env.PORT || config.get("port");

app.listen(PORT, () => {
  console.log(`Server is up on port ${PORT}`);
});
```

### config folder

- Create a folder `config` in the root of the project with a file called `default.json` in it:

```js
{
  "port": 5000
}
```

### models folder

- Create a folder `models` in the root of the project with a file called `value.js` in it:

```js
const Joi = require("@hapi/joi");

const validateValue = (value) => {
  const schema = Joi.object({
    amount: Joi.number().min(1).max(1000).required(),
  });

  return (
    schema.validate(value).error &&
    schema.validate(value).error.details[0].message
  );
};

exports.validateValue = validateValue;
```

### routes folder

- Create a folder `routes` in the root of the project and create a file for each routes (like `values.js`) in it:

```js
const { validateValue } = require("../models/value");
const express = require("express");
const router = express.Router();

const values = [
  { id: 1, amount: 1 },
  { id: 2, amount: 2 },
  { id: 3, amount: 3 },
];

router.get("/", (req, res) => {
  res.send(values);
});

router.get("/:id", (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  //Be careful that the req.params.id is string. So if you want to use it like an integer: paresInt(req.params.id)
  if (!value) {
    return res.status(404).send("Value not found");
  }

  res.send(value);
});

router.post("/", (req, res) => {
  const error = validateValue(req.body);
  if (error) {
    return res.status(400).send(error);
  }

  const value = {
    id: values.length + 1,
    amount: req.body.amount,
  };
  values.push(value);
  res.send(value);
});

router.put("/:id", (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  if (!value) {
    return res.status(404).send("Value not found");
  }

  const error = validateValue(req.body);
  if (error) {
    return res.status(400).send(error);
  }

  value.amount = req.body.amount;
  res.send(value);
});

router.delete("/:id", (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  if (!value) {
    return res.status(404).send("Value not found");
  }

  const index = values.indexOf(value);
  values.splice(index, 1); //at position index, remove 1 item

  res.send(value);
});

router.get("/:year/:id", (req, res) => {
  res.send({ params: req.params, query: req.query }); // to access route parameters and query strings
});

module.exports = router;
```

### startup folder

- Create a folder `startup` in the root of the project and create a file called `routes.js`:

```js
const express = require("express");
const valuesRouter = require("../routes/values");

module.exports = function (app) {
  app.use(express.json()); //It is a middleware. If req body has a json in it, it parses it.
  app.use("/api/values", valuesRouter); //This `/api` is for routing with nginx.
};
```

### Running the express app

```dos
nodemon index.js
```

If we hit this url in the browser `http://localhost:5000/api/values/45/2?sortBy=year`, we get:

```json
{
  "params": {
    "year": "45",
    "id": "2"
  },
  "query": {
    "sortBy": "year"
  }
}
```

## Middlewares

- An express app is a bunch of middleware functions called in sequence.
- When a request comes in, it goes through a series of middleware functions; such as `json()`, `route()`.
- Each middleware either terminates by sending a response to the client or gives control to the next middleware.
- Create a folder in the root of the application called `middlewares` with a file `validate.js` in it:

```js
module.exports = (validator) => {
  return (req, res, next) => {
    const error = validator(req.body);
    if (error) return res.status(400).send(error);
    next(); //If we omit this, our request will end up hanging.
  };
};
```

- Now, re-write the routes to use this middleware:

```js
const validate = require("../middlewares/validate");
const { validateValue } = require("../models/value");
const express = require("express");
const router = express.Router();

const values = [
  { id: 1, amount: 1 },
  { id: 2, amount: 2 },
  { id: 3, amount: 3 },
];

router.get("/", (req, res) => {
  res.send(values);
});

router.get("/:id", (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  if (!value) {
    return res.status(404).send("Value not found");
  }

  res.send(value);
});

router.post("/", validate(validateValue), (req, res) => {
  const value = {
    id: values.length + 1,
    amount: req.body.amount,
  };
  values.push(value);
  res.send(value);
});

router.put("/:id", validate(validateValue), (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  if (!value) {
    return res.status(404).send("Value not found");
  }

  value.amount = req.body.amount;
  res.send(value);
});

router.delete("/:id", (req, res) => {
  const value = values.find((value) => value.id === parseInt(req.params.id));
  if (!value) {
    return res.status(404).send("Value not found");
  }

  const index = values.indexOf(value);
  values.splice(index, 1);

  res.send(value);
});

router.get("/:year/:id", (req, res) => {
  res.send({ params: req.params, query: req.query });
});

module.exports = router;
```

### Global error middleware

- In the `middlewares` folder, add `error.js` file:

```js
module.exports = function error(err, req, res, next) {
  console.error(err.message, err);

  res.status(500).send("Something failed.");
};
```

- Add this middleware to the `routes.js` in `startup` folder:

```js
const express = require("express");
const valuesRouter = require("../routes/values");
const error = require("../middlewares/error");

module.exports = function (app) {
  app.use(express.json());
  app.use("/api/values", valuesRouter);
  app.use(error); // this should be after all routes
};
```

- Install `express-async-errors` package.
- In `startup` folder create `logging.js`:

```js
require("express-async-errors");

module.exports = function () {
  process.on("uncaughtException", (ex) => {
    console.error(ex.message, ex);
    process.exit(1);
  });

  process.on("unhandledRejection", (ex) => {
    console.error(ex.message, ex);
    process.exit(1);
  });
};
```

- We could use `winston`, which is a very popular logging package:

```dos
npm i winston winston-mongodb
```

```js
const winston = require("winston");
require("winston-mongodb");
require("express-async-errors");

module.exports = function () {
  process.on("uncaughtException", (ex) => {
    winston.error(ex.message, ex);
    process.exit(1);
  });

  process.on("unhandledRejection", (ex) => {
    winston.error(ex.message, ex);
    process.exit(1);
  });

  winston.add(winston.transports.File, {
    filename: "logfile.log",
    level: "info", // info and above
  });
  winston.add(winston.transports.MongoDB, {
    db: "mongodb://localhost/videostore",
    level: "error", // only errors
  });
};
```

- In the `app.js`:

```js
const express = require("express");
const app = express();

require("./startup/logging")();
require("./startup/routes")(app);

module.exports = app;
```

### Serve static files

- Use this middleware to serve static files from a certain path on the server:

```js
app.use(express.static("client/build"));
```

### Third-party middlewares

- `helmet` is used to secure application by setting various HTTP headers.
- `compression` is used to compress the http response sent to client.
- `morgan` is used to log http requests on the console, file, database,... To use it, in the `app.js`, we write code like this:

```js
const express = require("express");
const app = express();

require("./startup/logging")();
require("./startup/routes")(app);

if (app.get("env") === "development") {
  app.use(morgan("tiny"));
  console.log("Morgan enabled...");
}

module.exports = app;
```

It should be noted that `process.env.NODE_ENV` is essentially the same as `app.get("env")`, but if we haven't set the `process.env.NODE_ENV`, `process.env.NODE_ENV` returns undefined whereas `app.get("env")` returns `development`.

## Configurations in different environments

- We can have `production.json`, `development.json`, and `test.json` in the config folder to override the settings in `default.json`.
- We don't write secrets in these files. We create another file `custom-environment-variables.json` and map configuration settings to env variables:

```json
{
  "mail": {
    "password": "app_password"
  }
}
```

- So we just write `config.get('mail.host')` in the code, if it is in the config file (corresponding to the current env), it will read that. If it's not there, then it will look at `custom-environment-variables.json`.

## Debugging

With this package, we don't have to remove out `console.log`s. We can have different types of debuggers and we can control them by environment variables.

```dos
npm i debug
```

```js
var dbDebugger = require("debug")("app:db");
var startupDebugger = require("debug")("app:startup");

// Db work...
dbDebugger("Connected to the database...");
```

- If we set `DEBUG` to nothing, we don't see anything.
- If we set `DEBUG` env variable to `app:db`, we only see the debug messages for `dbDebugger`.
- If we set `DEBUG` to `app:db,app:startup`, we see both debugging messages.
- If we set `DEBUG` to `app:*`, we see all debugging messages.

- Shortcut to set the env variable and run the app at the same time:

```dos
DEBUG=app:db nodemon index.js
```

## Templating Engines

There are different template engines such as `pug`, `hbs`, `mustache`, and `ejs`.

# MongoDB, Mongoose, and Redis

## MongoDB and Mongoose

- In `NoSQL`, we have `collections` and `documents` instead of tables and records/rows.
- In `mongoose`, we use `schema` to define the shape of documents in a `mongoDb` collection.

### Creating the db

- In `https://cloud.mongodb.com/`, create a new project and a cluster for it.
- Click on `CONNECT` and add a new database user.
- Add a whitelist ip address (0.0.0.0/0).
- Choose a connection method and then click on `Connect your application`.
- Copy the string: `mongodb+srv://admin:<password>@sample-f7ktv.mongodb.net/test?retryWrites=true&w=majority`.
- Replace `<password>` with the database user password.
- Replace `test` with the name of database.

### Connecting to the db

```dos
npm i mongoose
```

- In `db.js` file in the `startup` folder:

```js
const mongoose = require("mongoose");
const config = require("config");

module.exports = function () {
  if (!config.get("db")) {
    throw new Error("MongoURI was not supplied.");
  }

  mongoose
    .connect(config.get("db"), {
      useNewUrlParser: true,
      useCreateIndex: true,
      useFindAndModify: false,
      useUnifiedTopology: true,
    })
    .then(() => console.log(`Connected to db...`));
};
```

- In the `app.js`, add `require('./startup/db')();` after `require("./startup/routes")(app);`.
- In the `default.json`, add `"db": "mongodb+srv://admin:Pa$$w0rd@sample-f7ktv.mongodb.net/sample?retryWrites=true&w=majority",`.

### Creating a schema

- In the `models` folder, in the `value.js` file:

```js
const Joi = require("@hapi/joi");
const mongoose = require("mongoose");

const valueSchema = new mongoose.Schema(
  {
    amount: { type: Number, required: true },
  },
  {
    timestamps: true,
  }
);

const Value = mongoose.model("Value", valueSchema); //The first arg is the singular name of the collection in the db.

const validateValue = (value) => {
  const schema = Joi.object({
    amount: Joi.number().min(1).max(1000).required(),
  });

  return (
    schema.validate(value).error &&
    schema.validate(value).error.details[0].message
  );
};

module.exports = { Value, validateValue };
```

Let's make a `playground.js` file:

```js
const mongoose = require("mongoose");

mongoose
  .connect(
    "mongodb+srv://admin:Pa$$w0rd@sample-f7ktv.mongodb.net/playground?retryWrites=true&w=majority",
    {
      useNewUrlParser: true,
      useCreateIndex: true,
      useFindAndModify: false,
      useUnifiedTopology: true,
    }
  )
  .then(() => console.log(`Connected to the db...`));

const courseSchema = new mongoose.Schema({
  name: { type: String },
  author: { type: String },
  price: { type: Number },
  tags: [{ type: String }],
  date: { type: Date, default: Date.now }, //Because it has default, we don't have to specify a date
  isPublished: { type: Boolean },
});

const Course = mongoose.model("Course", courseSchema);
```

### Create and query

- Before saving a document (running `save` method on it), its `isNew` property is true.

```js
const createCourse = async () => {
  const course = new Course({
    name: "React2 Course",
    author: "John",
    price: 18,
    tags: ["node", "frontend"],
    isPublished: true,
  });

  const course = await course.save();
  console.log(course);
};

const getCourses = async () => {
  const courses = await Course.find(); //Retrieves all courses
  console.log(courses);
};

const getCoursesWithFilter1 = async () => {
  const courses = await Course.find({ author: "Ben", isPublished: true });
  console.log(courses);
};

const getCoursesWithFilterPaginationAndSorting = async (page, size) => {
  const courses = await Course.find({ author: "Ben" })
    .skip((page - 1) * size)
    .limit(size)
    .sort({ name: 1 }) //1 means ascending and -1 means descending. Alternatively, you could use sort('name') and sort('-name'). Usally, sort must be before skip and limit.
    .select({ name: 1, tags: 1 }); //only returns name and tags. Alternatively, you could use select('name tags').

  console.log(courses);
};

const getCoursesWithComparison = async () => {
  const courses = await Course.find({
    author: "Ben",
    price: { $gt: 20, $lte: 55 }, //$eq, $ne (not equal), $gt, $gte (greater than or equal to), $lt, $lte (less than or equal to), $in, $nin (not in)
  });

  console.log(courses);
};

const getCoursesWithLogicalOperators = async () => {
  const courses = await Course.find()
    .or([{ author: "Ben" }, { isPublished: false }])
    .and([{ price: { $gt: 10 } }]); //it is the same as passing the query object to the find method

  console.log(courses);
};

const getCoursesWithFilter2 = async () => {
  const courses = await Course.find({ tags: "backend" }); //To have 'backend' inside tags array (not very intuitive).
  console.log(courses);
};

const getCoursesWithFilter3 = async () => {
  const courses = await Course.find({ tags: ["node", "backend"] }); //To be exactly like this (even the order is important).
  console.log(courses);
};

const getCoursesWithFilter4 = async () => {
  const courses = await Course.find().and([
    //To have 'node' and 'frontend' in any order and maybe more tags.
    { tags: "frontend" },
    { tags: "node" },
  ]);
  console.log(courses);
};

const getCoursesWithFilter5 = async () => {
  const courses = await Course.find().or([
    { tags: "frontend" },
    { tags: "backend" },
  ]);
  console.log(courses);
};

const getCoursesWithFilter6 = async () => {
  const courses = await Course.find({
    tags: { $in: ["frontend", "backend"] },
  }); //exactly like above
  console.log(courses);
};

const getCoursesWithRegex = async () => {
  //   const courses = await Course.find({ author: /^J/ }); //starts with J
  //   const courses = await Course.find({ author: /n$/ }); //ends with n
  //   const courses = await Course.find({ author: /N$/i }); //ends with n or N
  const courses = await Course.find({ author: /.*J.*/ }); //contains e somewhere

  console.log(courses);
};

const getCount = async () => {
  const count = await Course.find({ author: "Ben" }).count();

  console.log(count); //6
};
```

#### Example of a search query

```js
module.exports = (criteria, sortProperty, offset = 0, limit = 20) => {
  const query = Artist.find(buildQuery(criteria))
    .sort({ [sortProperty]: 1 })
    .skip(offset)
    .limit(limit);

  return Promise.all([query, Artist.find(buildQuery(criteria)).count()]).then(
    (results) => {
      return {
        all: results[0],
        count: results[1],
        offset: offset,
        limit: limit,
      };
    }
  );
};

const buildQuery = (criteria) => {
  const query = {};

  if (criteria.name) {
    query.$text = { $search: criteria.name };
  }

  if (criteria.age) {
    query.age = {
      $gte: criteria.age.min,
      $lte: criteria.age.max,
    };
  }

  if (criteria.yearsActive) {
    query.yearsActive = {
      $gte: criteria.yearsActive.min,
      $lte: criteria.yearsActive.max,
    };
  }

  return query;
};
```

#### Lean queries

- If you add `.lean()` at the end of a query, documents returned from queries with the lean option enabled are plain javascript objects, not Mongoose Documents. They have no save method, getters/setters, virtuals, or other Mongoose features. So its usually optimal for GET endpoints and .find() operations that don’t use .save() or virtuals.

```js
const docs = await Model.find().lean();
docs[0] instanceof mongoose.Document; // false
```

- Lean is great for high-performance, read-only cases.

#### Indexes

- MongoDB supports secondary indexes. With mongoose, we define these indexes within our Schema at the path level or the schema level. Defining indexes at the schema level is necessary when creating `compound indexes`.

```js
var animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true }, // field level
});

animalSchema.index({ name: 1, type: -1 }); // schema level
```

- The “1” or “-1” indicates the sort order of the properties. The order in which the fields are added to the index is important. For a more detailed.

- When your application starts up, Mongoose automatically calls createIndex for each defined index in your schema. Mongoose will call createIndex for each index sequentially, and emit an 'index' event on the model when all the createIndex calls succeeded or when there was an error. While nice for development, it is recommended this behavior be disabled in production since index creation can cause a significant performance impact. Disable the behavior by setting the autoIndex option of your schema to false, or globally on the connection by setting the option autoIndex to false.

```js
mongoose.connect("mongodb://user:pass@localhost:port/database", {
  autoIndex: false,
});
```

#### Run queries in parallel

- A common mistake I see in NodeJS code whenever people use async/await is that people run operations one after the other when they don’t need to be. For example:

```js
const user = new User({ name: "bob" });
const post = new Post({ title: "hello" });

await user.save();
await post.save();
```

- There’s no reason to wait until the user is saved in order to save the post. Instead, the database operations can be run in parallel using Promise.all as follows:

```js
const [user, post] = await Promise.all([user.save(), post.save()]);
```

### Update and delete

- You can update a document by using instance method of `update()` too. It sends an update command with this document \_id as the query selector. But using model methods are more common (the first method however is set and save which is also instance method):

```js
const updateCourseByQueryFirstApproach = async (id) => {
  const course = await Course.findById(id);
  if (!course) return;

  course.isPublished = true;
  course.author = "Nas";

  //   course.set({ //exactly like above
  //     isPublished: true,
  //     author: "Nas",
  //   });

  const result = await course.save();
  console.log(result);
};
updateCourseByQueryFirstApproach("5ea245a98e20d022b8782810");

const updateCourseByUpdateFirstApproach = async (id) => {
  const result = await Course.update(
    { _id: id },
    {
      $inc: { price: 10 },
      $set: {
        author: "Ben",
        isPublished: false,
      },
    }
  );

  console.log(result);
  //   {
  //     n: 1,
  //     nModified: 1,
  //     opTime: {
  //       ts: Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1587697892 },
  //       t: 3
  //     },
  //     electionId: 7fffffff0000000000000003,
  //     ok: 1,
  //     '$clusterTime': {
  //       clusterTime: Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1587697892 },
  //       signature: { hash: [Binary], keyId: [Long] }
  //     },
  //     operationTime: Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1587697892 }
  //   }
};
updateCourseByUpdateFirstApproach("5ea249dc6ebb361538f1418d");

const updateCourseByUpdateFirstApproachAndReturnCourse = async (id) => {
  const course = await Course.findByIdAndUpdate(
    id,
    {
      $inc: { price: 10 },
      $set: {
        author: "Ben",
        isPublished: false,
      },
    },
    {
      new: true, //without this, it returns the course before update
    }
  );

  console.log(course);
};
updateCourseByUpdateFirstApproachAndReturnCourse("5ea249dc6ebb361538f1418d");

- You can delete a document by using instance method of `remove()` too. But using model methods are more common:

const removeCourse = async (id) => {
  //   const result1 = await Course.deleteOne({ _id: id }); //if the filter returns more than one result, it deletes the first match.
  //   const result2 = await Course.deleteMany({ author: 'John' });
  //   const course = await Course.findByIdAndRemove(id);
  const course = await Course.findOneAndDelete({ _id: id }); //You should use findOneAndDelete() unless you have a good reason not to.

  //   console.log(result);
  //   {
  //     n: 1,
  //     nModified: 1,
  //     opTime: {
  //       ts: Timestamp { _bsontype: 'Timestamp', low_: 4, high_: 1587698338 },
  //       t: 3
  //     },
  //     electionId: 7fffffff0000000000000003,
  //     ok: 1,
  //     '$clusterTime': {
  //       clusterTime: Timestamp { _bsontype: 'Timestamp', low_: 4, high_: 1587698338 },
  //       signature: { hash: [Binary], keyId: [Long] }
  //     },
  //     operationTime: Timestamp { _bsontype: 'Timestamp', low_: 4, high_: 1587698338 }
  //   }

  console.log(course);
  //   {
  //     tags: [ 'node', 'frontend' ],
  //     _id: 5ea247e8d8cd5c28b4316003,
  //     name: 'React Course',
  //     author: 'Ben',
  //     isPublished: false,
  //     date: 2020-04-24T01:59:04.310Z,
  //     __v: 0
  //   }
};
removeCourse("5ea247e8d8cd5c28b4316003");
```

### Simple sample

```js
const { Value } = require("../models/value");
const validate = require("../middlewares/validate");
const { validateValue } = require("../models/value");
const express = require("express");
const router = express.Router();

router.get("/", async (req, res) => {
  const values = await Value.find();

  res.send(values);
});

router.get("/:id", async (req, res) => {
  const value = await Value.findById(req.params.id);

  if (!value) {
    return res.status(404).send("Value not found");
  }

  res.send(value);
});

router.post("/", validate(validateValue), async (req, res) => {
  const value = new Value({
    amount: req.body.amount,
  });

  await value.save();

  res.send(value);
});

router.put("/:id", validate(validateValue), async (req, res) => {
  const value = await Value.findByIdAndUpdate(
    req.params.id,
    { amount: req.body.amount },
    { new: true }
  );

  if (!value) {
    return res.status(404).send("Value not found");
  }

  res.send(value);
});

router.delete("/:id", async (req, res) => {
  const value = await Value.findByIdAndDelete(req.params.id);

  if (!value) {
    return res.status(404).send("Value not found");
  }

  res.send(value);
});

router.get("/:year/:id", (req, res) => {
  res.send({ params: req.params, query: req.query });
});

module.exports = router;
```

### Validation

- Apart from `joi` which validates what the client sends us, we want to have validation for mongoose.
- Change the schema to:

```js
const courseSchema = new mongoose.Schema({
  name: { type: String, required: true }, //this validation is only for mongoose. MongoDB doesn't care.
  author: { type: String },
  price: { type: Number },
  tags: [{ type: String }],
  date: { type: Date, default: Date.now },
  isPublished: { type: Boolean },
});
```

Now, if we want to save invalid data, we will get an exception:

```js
const createCourse = async () => {
  const course = new Course({
    // author: "John",
    price: 18,
    tags: ["node", "frontend"],
    isPublished: true,
  });

  try {
    const course = await course.save();
    console.log(course);
  } catch (ex) {
    console.log(ex.message);
  }
};
createCourse();
```

#### Built-in and custom validators in mongoose

- we can also pass a function that results in a boolean to `required` validator.
- we can use `validate` method to add custom validations (normal or async).
- Apart from validators, we have some options such as `trim`, `lowercase`, `get`, `set`, ...

```js
const courseSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
    match: /^c/,
    trim: true,
    lowercase: true,
  },
  author: {
    type: String,
    minlength: 5,
    maxlength: 20,
    trim: true,
    uppercase: true,
  },
  category: {
    type: String,
    enum: ["web", "mobile", "network"], //enum is for strings.
  },
  price: {
    type: Number,
    min: 1, //We have min and max for dates too.
    max: 100,
    required() {
      //We can't use arrow function here.
      return this.isPublished;
    },
    get: (v) => Math.round(v), //Intercepts when retrieving from db.
    set: (v) => Math.round(v), //Intercepts when writing to db.
  },
  tags: {
    type: [String],
    validate(value) {
      if (!(value && value.length > 0)) {
        throw new Error("A course should have at least one tag");
      }
    },
  },
  date: { type: Date, default: Date.now },
  isPublished: {
    type: Boolean,
    async validate(value) {
      const courses = await Course.find(); //An example of async work. Doesn't mean anything here.
      if (false) {
        throw new Error("This is an async validation");
      }
    },
  },
});
```

### Relationships

- There is a trade-off between `query performance` (using embedded docs) vs `consistency` (using references).

#### Using references

```js
const authorSchema = new mongoose.Schema({
  name: { type: String, required: true },
  age: { type: Number },
});
authorSchema.virtual("courses", {
  //This is completely optional. This is like nav props. But it is very useful to get deeply nested objects (see below).
  ref: "Course",
  localField: "_id",
  foreignField: "author",
});
const Author = mongoose.model("Author", authorSchema);

const courseSchema = new mongoose.Schema({
  title: { type: String, required: true },
  price: { type: Number, required: true },
  isPublished: { type: Boolean, default: false },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    required: true,
    ref: "Author",
  },
});
const Course = mongoose.model("Course", courseSchema);

const createAuthor = async () => {
  const author = new Author({
    name: "John",
    age: 35,
  });

  try {
    const result = await author.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
  }
};
// createAuthor();

const createCourse = async () => {
  const course = new Course({
    title: "course1",
    price: 10,
    author: "5ea2cffaee7d71222c93facc",
  });
  // it doesn't have to be id, we could create the course without the author and then: course.author = author; or just pass author to the above. mongoose is smart enough to pull the id out of ref property.

  try {
    const result = await course.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
  }
};
// createCourse();

const getCourse = async () => {
  const course = await Course.findById("5ea2d00e9577a934f0a14484").populate(
    "author", // this author is the property of courseSchema and not the author model.
    "name -_id" //only include name and exclude _id
  ); //We also can have multiple populate('category', 'name')

  console.log(course);
};
getCourse();
// {
//   isPublished: false,
//   _id: 5ea2d00e9577a934f0a14484,
//   title: 'course1',
//   price: 10,
//   author: { name: 'John' },
//   __v: 0
// }

const getAuthorAndCourses = async () => {
  const author = await Author.findById("5ea2cffaee7d71222c93facc");
  console.log(author.courses); //undefined

  await author.populate("courses").execPopulate(); //with that virtual property.
  console.log(author.courses);
  // [
  // {
  //     isPublished: false,
  //     _id: 5ea2d00e9577a934f0a14484,
  //     title: 'course1',
  //     price: 10,
  //     author: 5ea2cffaee7d71222c93facc,
  //     __v: 0
  // }
  // ]
};
getAuthorAndCourses();
```

- If it was deeply nested and we want to load everything in one go:

```js
const author = await Author.findOne({ name: "Ben" }).populate({
  path: "posts",
  populate: {
    path: "comments",
    model: "Comment",
    populate: {
      path: "author", //this is comment author
      model: "Author",
    },
  },
});
```

- In fact, there is no real association for forcing referential integrity.

#### Using embedded documents

- Sub-document: with defining a new schema, each will have its own unique id.

##### Storing the `one side` in the `many side`

- In this way, we only have `course` document and if we want to update the `author`, we should `course.author.name = "another name";` and then `course.save();`.

```js
const courseSchema = new mongoose.Schema({
  title: { type: String, required: true },
  price: { type: Number, required: true },
  isPublished: { type: Boolean, default: false },
  author: {
    type: new mongoose.Schema({
      name: { type: String, required: true },
      age: { type: Number },
    }),
    required: true,
  },
});
const Course = mongoose.model("Course", courseSchema);

const createCourse = async () => {
  const course = new Course({
    title: "course1",
    price: 10,
    author: {
      name: "Ben",
      age: 34,
    },
  });

  try {
    const result = await course.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
  }
};
createCourse();
```

##### Storing the `many side` in the `one side`

- We don't store `many side` in the `one side` in case that the size of the document becomes larger than 4mb. Also, in case that we want to query the many side separately, this way makes it really hard (on the other hand, listing all the courses of an author would be very easy.).
- If we want to add another course later, we can load the author on memory and push another course in `author.courses.push(course);` and then `author.save();`.
- For removing a course, `const course = author.courses.id(courseId);`, then `course.remove();`, and finally `author.save();`.

```js
const authorSchema = new mongoose.Schema(
  {
    name: { type: String, required: true },
    age: { type: Number },
    courses: [
      new mongoose.Schema({
        title: { type: String, required: true },
        price: { type: Number, required: true },
        isPublished: { type: Boolean, default: false },
      }),
    ],
  },
  {
    timestamps: true,
  }
);
authorSchema.virtual("courseCount").get(function () {
  // defining a virtual getter. This virtual field is only present on mongoose and won't be saved to the MongoDB.
  return this.courses.length;
});
const Author = mongoose.model("Author", authorSchema);

const createAuthor = async () => {
  const author = new Author({
    name: "Ben",
    age: 34,
    courses: [
      {
        title: "course1",
        price: 10,
      },
      {
        title: "course1",
        isPublished: true,
        price: 29,
      },
    ],
  });

  try {
    const result = await author.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
    //     {
    //   _id: 5ea278f6cb57b207705744dd,
    //   name: 'Ben',
    //   age: 34,
    //   courses: [
    //     {
    //       isPublished: false,
    //       _id: 5ea278f6cb57b207705744de,
    //       title: 'course1',
    //       price: 10
    //     },
    //     {
    //       isPublished: true,
    //       _id: 5ea278f6cb57b207705744df,
    //       title: 'course1',
    //       price: 29
    //     }
    //   ],
    //   createdAt: 2020-04-24T05:28:23.002Z,
    //   updatedAt: 2020-04-24T05:28:23.002Z,
    //   __v: 0
    // }
  }
};
createAuthor();
```

If you wanted to update one course in this scenario:

```js
const publishCourseInAuthor = async (authorId, title) => {
  await Author.updateOne(
    // ATTENTION: author document is very big (because of all its sub-docs) so never try to fetch one document and then update it!
    {
      _id: authorId,
      courses: {
        // Because courses is an array, $elemMatch is used to go through all its elements and find this match. so the whole query is to find an author which has a course with that condition.
        $elemMatch: { title: title, isPublished: false },
      },
    },
    {
      $set: { "courses.$.isPublished": true }, // $ is lined up with the $elemMatch. The positional $ operator variable, which holds the "matched" position in the array.
    }
  );
};
publishCourseInAuthor("5ea2cb3ca322c82ce8441b81", "course1");
```

#### Using hybride

```js
const authorSchema = new mongoose.Schema({
  name: { type: String, required: true },
  age: { type: Number },
});
const Author = mongoose.model("Author", authorSchema);

const courseSchema = new mongoose.Schema({
  title: { type: String, required: true },
  price: { type: Number, required: true },
  isPublished: { type: Boolean, default: false },
  author: {
    _id: {
      type: mongoose.Schema.Types.ObjectId,
      required: true,
      ref: "Author",
    },
    name: { type: String, required: true },
  },
});
const Course = mongoose.model("Course", courseSchema);

const createAuthor = async () => {
  const author = new Author({
    name: "Beni",
    age: 40,
  });

  try {
    const result = await author.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
  }
};
// createAuthor();

const createCourse = async () => {
  const course = new Course({
    title: "course1",
    price: 10,
    author: {
      _id: "5ea27fda483a4717b48ca9bf",
      name: "Beni",
    },
  });

  try {
    const result = await course.save();
    console.log(result);
  } catch (ex) {
    console.log(ex.message);
  }
};
createCourse();
```

### Transactions

- We can use `Two Phase Commits` technique to perform a transaction in mongoDb or use a package called `fawn`.

```js
const Fawn = require("fawn");

Fawn.init(mongoose);

try {
  new Fawn.Task() // this library defines a task (=transaction) which means all the followings or none
    .save("rentals", rental)
    .update("movies", { _id: movie._id }, { $inc: { numberInStock: -1 } })
    .run();

  res.send(rental);
} catch (ex) {
  res.status(500).send("Something failed.");
}
```

### Object Id

- ObjectIDs are generated by MongoDB driver () and are used to uniquely identify a document. They consist of 12 bytes:
  - 4 bytes: timestamp
  - 3 bytes: machine identiﬁer
  - 2 bytes: process identiﬁer
  - 3 byes: counter
- ObjectIDs are almost unique. In theory, there is a chance for two ObjectIDs to be equal but the odds are very low (1/16,000,000) for most real-world applications.

```js
const mongoose = require("mongoose");

const id = new mongoose.Types.ObjectId();
console.log(id); //5ea2973c053be822205c56d7
console.log(id.getTimestamp()); //2020-04-24T07:37:32.000Z
console.log(mongoose.Types.ObjectId.isValid("1234")); //false
console.log(mongoose.Types.ObjectId.isValid(id)); //true
```

- For validation of ObjectID input:

```dos
npm i joi-objectid
```

```js
const Joi = require("@hapi/joi");
Joi.objectId = require("joi-objectid")(Joi);

const schema = Joi.object({
  id: Joi.objectId(),
  //...
});
```

### Middlewares

- When defining mongoose schema, we have different events that we can define `pre` or `post` hooks for them. The example of these events are `init`, `validate`, `save` and `remove`.

![](/md/101.jpg)

```js
userSchema.pre("remove", async function (next) {
  const user = this;

  const Stuff = mongoose.model("Stuff"); // it is better to use mongoose.model instead of importing Stuff model. Because if we had another middleware in Stuff model that needed User model, we would have cyclic dependency.

  await Stuff.deleteMany({ owner: user._id });

  next();
});
```

## Redis

- Is an in-memory key/value store database used for caching.
- Data types in redis:
  - _strings_
  - _lists_: lists of strings
  - _sets_: un-ordered and non-repeating collections of strings
  - _sorted sets_
  - _hashes_: key/value pairs. They are like js objects.
  - ...
- It has both caching and persistence capabilities.

### Setup

- Install the redis server on your local computer from here `https://github.com/microsoftarchive/redis/releases`.
- Install redis in your node project:

```dos
npm i redis
```

### String methods

Node Redis currently doesn't natively support promises (this is coming in v4), however you can wrap the methods you want to use with promises using the built-in Node.js `util.promisify` method.

```js
const redis = require("redis");
const { promisify } = require("util");

const redisClient = redis.createClient();

redisClient.get = promisify(redisClient.get);
redisClient.exists = promisify(redisClient.exists);
redisClient.ttl = promisify(redisClient.ttl);

const exec = async () => {
  redisClient.set("key1", 100);
  console.log(await redisClient.get("key1")); //100

  redisClient.incr("key1");
  console.log(await redisClient.get("key1")); //101

  redisClient.decr("key1");
  console.log(await redisClient.get("key1")); //100

  console.log(await redisClient.exists("key1")); //1
  console.log(await redisClient.exists("key2")); //0

  redisClient.del("key1");
  console.log(await redisClient.exists("key1")); //0
  console.log(await redisClient.get("key1")); //null

  redisClient.set("key1", 100);
  redisClient.flushall(); //wipes out everything
  console.log(await redisClient.exists("key1")); //0
  console.log(await redisClient.get("key1")); //null

  redisClient.set("key1", 100);
  redisClient.expire("key1", 10); //expires in 10 seconds
  setTimeout(async () => {
    console.log(await redisClient.ttl("key1")); //9
  }, 1000);

  redisClient.setex("key2", 1, 100); //key seconds value
  setTimeout(async () => {
    console.log(await redisClient.ttl("key2")); //-2: means expired
  }, 1500);

  redisClient.mset("key3", "Hello", "Key4", "World"); //setting multiple key value pairs together
  console.log(await redisClient.get("key3")); //Hello
  console.log(await redisClient.get("Key4")); //World
};
exec();
```

### List methods

```js
const redis = require("redis");
const { promisify } = require("util");

const redisClient = redis.createClient();

redisClient.lrange = promisify(redisClient.lrange);
redisClient.llen = promisify(redisClient.llen);

const exec = async () => {
  redisClient.flushall();

  redisClient.lpush("key1", "value1");
  redisClient.lpush("key1", "value2");
  redisClient.lpush("key1", "value3");
  redisClient.rpush("key1", "value4");
  console.log(await redisClient.lrange("key1", 0, -1)); //[ 'value3', 'value2', 'value1', 'value4' ]
  console.log(await redisClient.llen("key1")); //4

  redisClient.lpop("key1");
  redisClient.rpop("key1");
  console.log(await redisClient.lrange("key1", 0, -1)); //[ 'value2', 'value1' ]

  redisClient.linsert("key1", "AFTER", "value2", "value3");
  console.log(await redisClient.lrange("key1", 0, -1)); //[ 'value2', 'value3', 'value1' ]
};
exec();
```

### Set methods

```js
const redis = require("redis");
const { promisify } = require("util");

const redisClient = redis.createClient();

redisClient.sismember = promisify(redisClient.sismember);
redisClient.smembers = promisify(redisClient.smembers);
redisClient.scard = promisify(redisClient.scard);

const exec = async () => {
  redisClient.flushall();

  redisClient.sadd("cars", "Ford");
  redisClient.sadd("cars", "Honda");
  redisClient.sadd("cars", "BMW");

  console.log(await redisClient.sismember("cars", "Ford")); //1
  console.log(await redisClient.sismember("cars", "Volvo")); //0

  console.log(await redisClient.smembers("cars")); //[ 'Honda', 'BMW', 'Ford' ]

  console.log(await redisClient.scard("cars")); //3

  redisClient.srem("cars", "BMW");
  console.log(await redisClient.smembers("cars")); //[ 'Honda', 'Ford' ]
};
exec();
```

### Sorted set methods

```js
const redis = require("redis");
const { promisify } = require("util");

const redisClient = redis.createClient();

redisClient.zrank = promisify(redisClient.zrank);
redisClient.zrange = promisify(redisClient.zrange);

const exec = async () => {
  redisClient.flushall();

  redisClient.zadd("users", 34, "Ben");
  redisClient.zadd("users", 28, "Nas");
  redisClient.zadd("users", 28, "Kate");
  redisClient.zadd("users", 40, "Paul");

  console.log(await redisClient.zrank("users", "Ben")); //2 rank from low to high
  console.log(await redisClient.zrank("users", "Nas")); //1
  console.log(await redisClient.zrank("users", "Kate")); //0
  console.log(await redisClient.zrank("users", "Paul")); //3

  console.log(await redisClient.zrange("users", 0, -1)); //[ 'Kate', 'Nas', 'Ben', 'Paul' ]
};
exec();
```

### Hash methods

```js
const redis = require("redis");
const { promisify } = require("util");

const redisClient = redis.createClient();

redisClient.hget = promisify(redisClient.hget);
redisClient.hgetall = promisify(redisClient.hgetall);

const exec = async () => {
  redisClient.flushall();

  redisClient.hset("user:ben", "name", "Ben Abe"); //hashKey key value
  redisClient.hset("user:ben", "email", "Ben@test.com");

  console.log(await redisClient.hget("user:ben", "name")); //Ben Abe
  console.log(await redisClient.hget("user:ben", "email")); //Ben@test.com

  console.log(await redisClient.hgetall("user:ben")); //{ name: 'Ben Abe', email: 'Ben@test.com' }

  redisClient.hmset("user:john", "name", "John Doe", "email", "jdoe@test.com"); //hashKey, key1, val1, key2, val2
  console.log(await redisClient.hgetall("user:john")); //{ name: 'John Doe', email: 'jdoe@test.com' }

  redisClient.del("user:john");
  console.log(await redisClient.hgetall("user:john")); //null
};
exec();
```

### Publish and subscribe

- In the publisher module:

```js
const redis = require("redis");
const redisClient = redis.createClient({
  // host and port of redis server
});
const publisher = redisClient.duplicate();
//...
publisher.publish("whatever", data); //"whatever" is the channel.
```

- In the subscriber module:

```js
const redis = require("redis");
const redisClient = redis.createClient({
  // host and port of redis server
});
const subscriber = redisClient.duplicate();
//...
subscriber.on("message", (channel, message) => {
  //message is data
});
subscriber.subscribe("whatever"); //"whatever" is the channel.
```

### Implement caching in an app

- After requiring `mongoose` but before connecting, add this:

```js
require("../services/cache");
```

- In the `cache.js`:

```js
const config = require("config");
const mongoose = require("mongoose");
const redis = require("redis");
const { promisify } = require("util");

const client = redis.createClient(config.get("redisUrl"));
client.hget = promisify(client.hget);

const exec = mongoose.Query.prototype.exec; //It is a method that executes the query.

mongoose.Query.prototype.cache = function (options = {}) {
  //We hijacked Query class and added a method to it.
  // We didn't use inheritance because it was not easy to tell mongoose to use this extended class.
  this.useCache = true; // `this` refers to an instance of Query.
  this.hashKey = JSON.stringify(options.key || ""); // The definition of this variable is to target a subset of key value pairs in redis database to clear.
  return this; // To make it chain-able.
};

mongoose.Query.prototype.exec = async function () {
  //`exec` is a method of a query and returns the result. We want to intercept.
  if (!this.useCache) {
    return exec.apply(this, arguments);
  }

  const key = JSON.stringify(
    // We want the key to be the stringified version of the query itself plus the collection that the query is executing on.
    Object.assign({}, this.getQuery(), {
      // Instead of this, we could use spread operator.
      collection: this.mongooseCollection.name,
    })
  );

  const cacheValue = await client.hget(this.hashKey, key);

  if (cacheValue) {
    const doc = JSON.parse(cacheValue);
    return Array.isArray(doc)
      ? doc.map((d) => new this.model(d))
      : new this.model(doc); // We want to get the results from the cache, if there is one.
  }

  const result = await exec.apply(this, arguments); //Otherwise, we want to get the result and then store it in cache.

  client.hset(this.hashKey, key, JSON.stringify(result), "EX", 10); // Cache expires after 10 secs.

  return result;
};

module.exports = {
  clearHash(hashKey) {
    client.del(JSON.stringify(hashKey));
  },
};
```

- Add this middleware `cleanCache.js`:

```js
const { clearHash } = require("../services/cache");

module.exports = async (req, res, next) => {
  await next(); // with this trick, first the route handler is executed and then if it is a success, the cache wil be cleared
  clearHash(req.user.id);
};
```

- In the `blogRoutes.js` in the `routes` folder:

```js
app.get("/api/blogs/:id", requireLogin, async (req, res) => {
  const blog = await Blog.findOne({
    _user: req.user.id,
    _id: req.params.id,
  });

  res.send(blog);
});

app.get("/api/blogs", requireLogin, async (req, res) => {
  const blogs = await Blog.find({ _user: req.user.id }).cache({
    key: req.user.id, // if key is not provided, it will be empty string and all values will be flushed during clear
  });
  res.send(blogs);
});

app.post("/api/blogs", requireLogin, cleanCache, async (req, res) => {
  const { title, content, imageUrl } = req.body;

  const blog = new Blog({
    title,
    content,
    imageUrl,
    _user: req.user.id,
  });

  try {
    await blog.save();
    res.send(blog);
  } catch (err) {
    res.send(400, err);
  }
});
```

# Sorting, Filtering, and Pagination

```js
// GET /tasks?completed=true
// GET /tasks?limit=10&skip=20
// GET /tasks?sortBy=createdAt:desc
router.get("/tasks", auth, async (req, res) => {
  const match = {};
  const sort = {};

  if (req.query.completed) {
    match.completed = req.query.completed === "true";
  }

  if (req.query.sortBy) {
    const parts = req.query.sortBy.split(":");
    sort[parts[0]] = parts[1] === "desc" ? -1 : 1;
  }

  try {
    await req.user
      .populate({
        path: "tasks",
        match,
        options: {
          limit: parseInt(req.query.limit),
          skip: parseInt(req.query.skip),
          sort,
        },
      })
      .execPopulate();
    res.send(req.user.tasks);
  } catch (e) {
    res.status(500).send();
  }
});
```

# Authentication and Authorization

## Setting jwt private key

In `config.js` file in the `startup` folder:

```js
const config = require("config");

module.exports = function () {
  if (!config.get("jwtPrivateKey")) {
    throw new Error("FATAL ERROR: jwtPrivateKey is not defined.");
  }
};
```

- In the `app.js`, add `require("./startup/config")();` after `require("./startup/db")();`.
- In the `custom-environment-variables.json`, add `"jwtPrivateKey": "my_video_store_jwtPrivateKey",`.
- Set the `my_video_store_jwtPrivateKey` env variable on your machine.

## User schema

- Create the `user.js` file in the `models` folder:

```js
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const Joi = require("@hapi/joi");
const Stuff = require("./Stuff");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true,
    },
    email: {
      type: String,
      unique: true,
      required: true,
      trim: true,
      lowercase: true,
    },
    password: {
      type: String,
      required: true,
      minlength: 5,
      trim: true,
    },
    tokens: [
      {
        token: {
          type: String,
          required: true,
        },
      },
    ],
  },
  {
    timestamps: true,
  }
);

userSchema.methods.toJSON = function () {
  //not to send these info to the client
  const user = this;
  const userObject = user.toObject();

  delete userObject.password;
  delete userObject.tokens;

  return userObject;
};

// Creating an instance method
userSchema.methods.generateAuthToken = function () {
  // you cannot use arrow function to create a method as part of an object.
  const user = this; // `this` refers to the object that this method is called on.

  const token = jwt.sign(
    { _id: user._id.toString() },
    config.get("jwtPrivateKey")
  );

  user.tokens.push({ token });
  await user.save();

  return token;
};

userSchema.statics.findByCredentials = async (email, password) => {
  const user = await User.findOne({ email });

  if (!user) {
    return null;
  }

  const isMatch = await bcrypt.compare(password, user.password);

  if (!isMatch) {
    return null;
  }

  return user;
};

// Hash the plain text password before saving.
userSchema.pre("save", async function (next) {
  const user = this;

  if (user.isModified("password")) {
    const salt = await bcrypt.genSalt(10);
    user.password = await bcrypt.hash(user.password, salt);
  }

  next();
});

// Delete user related stuff when user is removed.
userSchema.pre("remove", async function (next) {
  const user = this;

  await Stuff.deleteMany({ owner: user._id });

  next();
});

const User = mongoose.model("User", userSchema);

const validateUser = (user) => {
  const schema = Joi.object({
    name: Joi.string().max(50).required(),
    email: Joi.string().max(255).required().email(),
    password: Joi.string()
      .min(5)
      .max(255) // this password is not hashed
      .required(),
  });

  return (
    schema.validate(user).error &&
    schema.validate(user).error.details[0].message
  );
};

const validateLogin = (user) => {
  const schema = Joi.object({
    email: Joi.string().max(255).required().email(),
    password: Joi.string().min(5).max(255).required(),
  });

  return (
    schema.validate(user).error &&
    schema.validate(user).error.details[0].message
  );
};

module.exports.validateUser = { User, validateUser, validateLogin };
```

## auth middleware

- Create `auth.js` in the `middlewares` folder:

```js
const jwt = require("jsonwebtoken");
const config = require("config");
const { User } = require("../models/user");

const auth = async (req, res, next) => {
  try {
    const token = req.header("Authorization").replace("Bearer ", "");
    const decoded = jwt.verify(token, config.get("jwtPrivateKey"));
    const user = await User.findOne({
      _id: decoded._id,
      "tokens.token": token,
    });

    if (!user) {
      throw new Error();
    }

    req.token = token;
    req.user = user;
    next();
  } catch (e) {
    res.status(401).send({ error: "Please authenticate." });
  }
};

module.exports = auth;
```

- We use this middleware for every routes that needs authentication.

## users route

- We need at least four routes:
  1. `POST /api/users` to register a user
  2. `POST /api/users/login` to login a user
  3. `POST /api/users/logout` to logout the user
  4. `GET /api/users/me` to get the current user
- Create the `users.js` route in the `routes` folder:

```js
router.post("/users", validate(validateUser), async (req, res) => {
  let user = await User.findOne({ email: req.body.email });
  if (user) return res.status(400).send("User already registered.");

  user = new User(req.body);

  await user.save();

  // const refreshToken = await user.createSession();
  const token = await user.generateAuthToken();

  // res.status(201).send({ user, token, refreshToken }); // we can send it in x-refresh-token and x-access-token headers as well.
  res.status(201).send({ user, token });
});

router.post("/users/login", validate(validateLogin), async (req, res) => {
  const user = await User.findByCredentials(req.body.email, req.body.password);
  if (!user) return res.status(400).send("Invalid email or password.");

  // const refreshToken = await user.createSession();
  const token = await user.generateAuthToken();

  // res.status(201).send({ user, token, refreshToken }); // we can send it in x-refresh-token and x-access-token headers as well.
  res.send({ user, token });
});

router.post("/users/logout", auth, async (req, res) => {
  req.user.tokens = req.user.tokens.filter((token) => {
    return token.token !== req.token;
  });

  await req.user.save();

  res.send("Logged out successfully.");
});

router.post("/users/logoutAll", auth, async (req, res) => {
  req.user.tokens = [];

  await req.user.save();

  res.send("Logged out from all devices successfully.");
});

router.get("/users/me", auth, async (req, res) => {
  res.send(req.user);
});
```

- Add the following in `startup/routes.js`:

```js
const express = require("express");
const usersRouter = require("../routes/users");
const valuesRouter = require("../routes/values");
const error = require("../middlewares/error");

module.exports = function (app) {
  app.use(express.json());
  app.use("/api/users", usersRouter);
  app.use("/api/values", valuesRouter);
  app.use(error);
};
```

## Authorization

- We have two options: `Role-based` or `Operation-based`, the latter is better.
- In both case, user model should have a property called, `roles: [...]`, or `operations: [...]`.
- Then, we will create another middleware for every role or operation, for example `admin` role (`admin.js`):

```js
module.exports = function admin(req, res, next) {
  if (!req.user.roles.includes("admin"))
    return res.status(403).send("Forbidden: Access is denied.");
  next();
};
```

- When this middleware is called after `auth` middleware, `req.user` is populated.

## Refresh Tokens (This section has not been executed so be aware of the code)

- Storing tokens in the database, is not a good thing. Because for each auth middleware call, we have to check it with database and more importantly, if anyone gets access to the database, he can impersonate anyone.
- The solution is short-lived (15 mins) access-token (JWT) and then a long-lived refresh-token (randomly generated string) in the database.
- When access-token expires, it asks for another access-token and sends the refresh-token. If the refresh-token is valid, the API sends a new access token. So if we delete the refresh token of a particular session, the user has to login again (when a thief steals a laptop, I will request to delete the refresh token).
- So the userSchema should have a `sessions` property which is an array of session objects which contains a refresh token and its expiry DateTime in the form of a unix timestamp (number):

```js
const userSchema = new mongoose.Schema(
    {
      ...
      sessions: [{
        token: {
          type: String,
          required: true,
        },
        expiresAt: {
          type: Number,
          required: true
        }
      }]});
```

- We should exclude sessions in `toJSON` method.
- We should pass `{ expiresIn: "15m" }` as the third argument in the generateAuthToken method.
- We should also add these methods:

```js
const crypto = require("crypto");

userSchema.methods.generateRefreshToken = function () {
  return new Promise((resolve, reject) => {
    crypto.randomBytes(64, (err, buf) => {
      if (!err) {
        const token = buf.toString("hex");

        return resolve(token);
      }
    });
  });
};

userSchema.methods.createSession = function () {
  const user = this;

  return user
    .generateRefreshToken()
    .then((refreshToken) => {
      return saveSessionToDB(user, refreshToken);
    })
    .then((refreshToken) => {
      return refreshToken;
    })
    .catch((e) => Promise.reject("Failed to save session to DB. \n" + e));
};

userSchema.statics.findByIdAndToken = async (id, refreshToken) => {
  const User = this;

  const user = await User.findOne({
    id,
    "sessions.token": refreshToken,
  });

  return user;
};

userSchema.statics.hasRefreshTokenExpired = (expiresAt) => {
  return expiresAt < Date.now() / 1000;
};

// helper method
const saveSessionToDB = (user, refreshToken) => {
  return new Promise((resolve, reject) => {
    const expiresAt = generateRefreshTokenExpiryTime();

    user.sessions.push({ token: refreshToken, expiresAt });

    user
      .save()
      .then(() => {
        return resolve(refreshToken);
      })
      .catch((e) => {
        reject(e);
      });
  });
};

// helper method
const generateRefreshTokenExpiryTime = () => {
  const daysUntilExpire = 10;
  const secsUntilExpires = daysUntilExpire * 24 * 60 * 60;
  return Date.now() / 1000 + secsUntilExpires;
};
```

- Then in the user's routes please see the comments in the routes above. But for refreshing the tokens, we add this route:

```js
router.get("/users/me/access-token", verifySession, (req, res) => {
  const token = await req.user.generateAuthToken();

  res.send({ token });
});
```

- The `verifySession.js` in the `middlewares` folder:

```js
const verifySession = async (req, res, next) => {
  try {
    const refreshToken = req.header("x-refresh-token");
    const token = req.header("Authorization").replace("Bearer ", "");
    const decoded = jwt.verify(token, config.get("jwtPrivateKey"));
    const user = await User.findByIdAndToken({
      id: decoded._id,
      refreshToken,
    });

    if (!user) {
      throw new Error();
    }

    req.user = user;

    const session = user.sessions.find(
      (session) => session.token === refreshToken
    );

    if (User.hasRefreshTokenExpired()) {
      throw new Error();
    }

    next();
  } catch (e) {
    res.status(400).send({
      error: "Make sure that the refresh token and token are correct.",
    });
  }
};

module.exports = auth;
```

- In the front-end, if we get 401, first we try to refresh our token and if that fails too then logout.

## Google Auth

```dos
npm i google-auth-library axios
```

- In the `https://console.developers.google.com/` click on the `Create OAuth client ID`.
- Choose `Web application`. Give it a name. And `http://localhost:5000/api/users/google/callback` as the `Authorized redirect URIs`.
- Click on create.
- Add `googleClientId`, `googleClientSecret`, and `googleRedirectUrl` to `default.json` or to the env variable and map configuration to env variable in the `custom-environment-variables.json` file:

```json
{
  "port": 5000,
  "db": "mongodb+srv://admin:Pa$$w0rd@sample-f7ktv.mongodb.net/sample?retryWrites=true&w=majority",
  "jwtPrivateKey": "veryVerySecret",
  "appUrl": "https://www.something.com",
  "googleClientId": "594451966121-lvoio6ia31c6begt8b3adiusq8ndctmu.apps.googleusercontent.com",
  "googleClientSecret": "5tHKL-6rGo_UG_NSpchIqlNt",
  "googleRedirectUrl": "http://localhost:5000/api/users/google/callback"
}
```

- Create `googleOauth.js` in the `services` folder:

```js
const { OAuth2Client } = require("google-auth-library");
const axios = require("axios");
const config = require("config");

const client = new OAuth2Client(
  config.get("googleClientId"),
  config.get("googleClientSecret"),
  config.get("googleRedirectUrl")
);

const getAuthorizeUrl = () => {
  return client.generateAuthUrl({
    access_type: "offline",
    scope: "https://www.googleapis.com/auth/userinfo.profile",
  });
};

const getUserProfileAndGoogleToken = async (code) => {
  const {
    tokens: { access_token },
  } = await client.getToken(code);

  const { data: profile } = await axios.get(
    `https://www.googleapis.com/oauth2/v1/userinfo?alt=json&access_token=${access_token}`
  );

  return { profile, googleToken: access_token };
};

const logout = async (googleToken) => {
  try {
    await client.revokeToken(googleToken);
  } catch (ex) {
    console.log(ex);
  }
};

module.exports = { getAuthorizeUrl, getUserProfileAndGoogleToken, logout };
```

- Create `user.js` in the models folder:

```js
const mongoose = require("mongoose");
const jwt = require("jsonwebtoken");
const config = require("config");

const userSchema = new mongoose.Schema(
  {
    googleId: {
      type: String,
      unique: true,
      required: true,
    },
    tokens: [
      {
        token: {
          type: String,
          required: true,
        },
        googleToken: {
          type: String,
          required: true,
        },
      },
    ],
    credits: { type: Number, default: 0 },
  },
  {
    timestamps: true,
  }
);

userSchema.methods.toJSON = function () {
  const user = this;
  const userObject = user.toObject();

  delete userObject.tokens;

  return userObject;
};

userSchema.methods.generateAuthToken = async function (googleToken) {
  const user = this;

  const token = jwt.sign(
    { _id: user._id.toString() },
    config.get("jwtPrivateKey")
  );

  user.tokens.push({ token, googleToken });

  await user.save();

  return token;
};

const User = mongoose.model("User", userSchema);

module.exports = { User };
```

- Create `auth` middleware exactly like before.
- Create `users.js` route:

```js
const express = require("express");
const config = require("config");
const {
  getAuthorizeUrl,
  getUserProfileAndGoogleToken,
  logout,
} = require("../services/googleOauth");
const auth = require("../middlewares/auth");
const { User } = require("../models/user");

const router = express.Router();

router.get("/google", (req, res) => {
  res.redirect(getAuthorizeUrl());
});

router.get("/google/callback", async (req, res) => {
  const { profile, googleToken } = await getUserProfileAndGoogleToken(
    req.query.code
  );

  let user = await User.findOne({ googleId: profile.id });

  // sign in
  if (user) {
    const token = await user.generateAuthToken(googleToken);
    return res.redirect(`${config.get("appUrl")}/?token=${token}`);
  }

  // sign up
  user = await new User({ googleId: profile.id }).save();
  const token = await user.generateAuthToken(googleToken);
  return res.redirect(`${config.get("appUrl")}/?token=${token}`);
});

router.post("/logout", auth, async (req, res) => {
  const { googleToken } = req.user.tokens.find(
    (token) => token.token === req.token
  );

  req.user.tokens = req.user.tokens.filter((token) => {
    return token.token !== req.token;
  });
  await req.user.save();

  logout(googleToken);

  res.send("Logged out successfully.");
});

router.get("/me", auth, (req, res) => {
  res.send(req.user);
});

module.exports = router;
```

- Add the route above in the `startup/routes.js`.
- In the browser, go to `http://localhost:5000/api/users/google`.

# File Upload

## Multer

- There is another famous package `formidable` but we use `multer`.
- Install `multer` and `sharp`:

```js
npm i multer sharp
```

- The following middleware (`upload.js`) processes a single file associated with the given form field, here "file", and populates `req.file`:

```js
const multer = require("multer");

const upload = multer({
  limits: {
    fileSize: 10000000,
  },
  fileFilter(req, file, cb) {
    if (!file.originalname.match(/\.(jpg|jpeg|png)$/)) {
      return cb(new Error("Please upload an image"));
    }

    cb(undefined, true);
  },
});

module.exports = upload.single("file");
```

- Now create a `upload.js` route:

```js
const fs = require("fs");
const express = require("express");
const sharp = require("sharp");
const upload = require("../middlewares/upload");
const router = express.Router();

router.post(
  "/",
  upload,
  async (req, res) => {
    const buffer = await sharp(req.file.buffer)
      .resize({ width: 250, height: 250 })
      .png()
      .toBuffer();

    fs.mkdir("./uploads", { recursive: true }, (err) => {
      if (err) throw err;
    });

    fs.writeFile(`./uploads/${Date.now()}.png`, buffer, function (err) {
      if (err) throw err;
    });

    res.send("Uploaded");
  },
  (error, req, res, next) => {
    res.status(400).send({ error: error.message });
  }
);

module.exports = router;
```

- Note that we had to add:

```js
,
  (error, req, res, next) => {
    res.status(400).send({ error: error.message });
  }
```

in order to show the errors thrown by `upload.js` middleware, such as file size and type.

- Now, add the route to the `startup/routes.js`:

```js
const express = require("express");
const valuesRouter = require("../routes/values");
const uploadRouter = require("../routes/upload");
const error = require("../middlewares/error");

module.exports = function (app) {
  app.use(express.json());
  app.use("/api/values", valuesRouter);
  app.use("/api/upload", uploadRouter);
  app.use(error);
};
```

## AWS

```dos
npm i aws-sdk uuid
```

- Add AWS `accessKeyId` and `secretAccessKey` key in the `default.json` or in the env variable and map configuration to env variable in the `custom-environment-variables.json` file:

```json
{
  "accessKeyId": "sdfsfgdfg",
  "secretAccessKey": "sdfsfgdfh"
}
```

- The idea is:

![](/md/aws.jpg)

- Create a `awsUpload.js` route:

```js
const uuid = require("uuid/v1");
const AWS = require("aws-sdk");
const config = require("config");
const express = require("express");
const router = express.Router();
const auth = require("../middlewares/auth");

const s3 = new AWS.S3({
  accessKeyId: config.get("accessKeyId"),
  secretAccessKey: config.get("secretAccessKey"),
});

router.get("/", auth, (req, res) => {
  s3.getSignedUrl(
    "putObject",
    {
      Bucket: "my-bucket-123",
      ContentType: "image/jpeg",
      Key: `${req.user.id}/${uuid()}.jpeg`,
    },
    (err, url) => {
      res.send({ key, url });
    }
  );
});

module.exports = router;
```

- Register the route in `startup/routes.js`.
- In redux actions:

```js
export const submitBlog = (values, file, history) => async (dispatch) => {
  const {
    data: { key, url },
  } = await axios.get("/api/awsUpload");

  await axios.put(url, file, {
    headers: {
      "Content-Type": file.type,
    },
  });

  const { data: blog } = await axios.post("/api/blogs", {
    ...values,
    imageUrl: key,
  });

  history.push("/blogs");
  dispatch({ type: FETCH_BLOG, payload: blog });
};
```

- In the upload form:

```js
const [file, setFile] = useState(null);

//...

const handleSubmit = (e) => {
  e.preventDefault();

  submitBlog(formValues, file, history);
};

return (
  <form onSubmit={handleSubmit}>
    <h5>Add an Image</h5>
    <input
      onChange={(e) => setFile(e.target.files[0])}
      type="file"
      accept="image/*"
    />
    <button>Upload</button>
  </form>
);
```

# Sending Email

```dos
npm i @sendgrid/mail
```

- Create an Api key in the `https://app.sendgrid.com/settings/api_keys` website.
- Add this key in the `default.json` or in the env variable and map configuration to env variable in the `custom-environment-variables.json` file:

```json
{
  "port": 5000,
  "db": "mongodb+srv://admin:Pa$$w0rd@sample-f7ktv.mongodb.net/sample?retryWrites=true&w=majority",
  "sg": "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"
}
```

- Create a folder `services` with a file `emails.js` in it:

```js
const config = require("config");
const sgMail = require("@sendgrid/mail");
const welcomeEmail = require("./emailTemplates/welcomeEmail");

sgMail.setApiKey(config.get("sg"));

const sendTextEmail = (email, name) => {
  sgMail.send({
    to: email,
    from: "ben@test.com",
    subject: "Thanks for choosing us!",
    text: `Welcome to the app, ${name}. Let me know how you get along with the app.`,
  });
};

const sendHTMLEmail = (email, name) => {
  sgMail.send({
    to: email,
    from: "ben@test.com",
    subject: "Thanks for choosing us!",
    html: welcomeEmail(name),
  });
};

module.exports = {
  sendTextEmail,
  sendHTMLEmail,
};
```

- Create a folder `emailTemplates` in the `services` folder, with a `welcomeEmail.js`:

```js
module.exports = (name) => {
  return `
    <html>
      <body>
        <div style="text-align: center;">
          <h3>Welcome to the app ${name}!</h3>
          <p>Let me know how you get along with the app.</p>
        </div>
      </body>
    </html>
  `;
};
```

- Import `const { sendTextEmail, sendHTMLEmail } = require("../services/emails");` wherever you want to use it and use it:

```js
sendTextEmail(req.email, req.name);
sendHTMLEmail(req.email, req.name);
```

## nodemailer

```dos
npm i nodemailer
```

```js
var nodemailer = require("nodemailer");

var transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: "youremail@gmail.com",
    pass: "yourpassword",
  },
});

var mailOptions = {
  from: "youremail@gmail.com",
  to: "myfriend@yahoo.com, myotherfriend@yahoo.com",
  subject: "Sending Email using Node.js",
  text: "That was easy!",
  // html: "<h1>Welcome</h1><p>That was easy!</p>", // Instead of text, you can send html
};

transporter.sendMail(mailOptions, function (error, info) {
  if (error) {
    console.log(error);
  } else {
    console.log("Email sent: " + info.response);
  }
});
```

## Webhook

- In `sendgrid` the `Event Webhook` setting controls webhook notifications for events, such as bounces, clicks, opens, and more. This setting allows these events to be POSTed to a URL of your choosing such as `https://john-mern.herokuapp.com/api/surveys/webhooks`.
- You have to enable it in `Settings > Mail Settings > Event Settings` and check the options that you want like `Clicked`.
- Then in some intervals, that url will be posted by some requests with the `req.body` in the form of:

```js
[
  {
    email: "john_e63@yahoo.com",
    event: "click",
    ip: "115.70.55.156",
    sg_event_id: "Alg4-yVkToCDkNgeXmFnRQ",
    sg_message_id:
      "Dr_7evwJQPGDxWnEjrPPOA.filter0132p1iad2-30461-5E19A785-1F.0",
    timestamp: 1578744563,
    url: "http://localhost:3000/api/surveys/5e19a782607b910ed41c3235/yes",
    url_offset: { index: 0, type: "html" },
    useragent:
      "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36",
  },
];
```

### Path-parser

```dos
npm i path-parser
```

```js
const { Path } = require("path-parser");

const path = new Path("/users/:id");

// Matching
path.test("/users/00123");
// {
//  id: "00123"
// }
```

### Lodash

```dos
npm i lodash
```

```js
const _ = require("lodash");

person = { name: "ben", age: 35, job: "developer" };

list = [
  { email: "ben1@test.com", responded: false, age: 10 },
  { email: "ben1@test.com", responded: false, age: 20 },
  { email: "ben2@test.com", responded: true },
  { email: "ben3@test.com", responded: false },
  undefined,
];

list2 = [
  { email: "ben1@test.com", responded: false, age: 10 },
  { email: "ben1@test.com", responded: false, age: 20 },
  { email: "ben2@test.com", responded: true },
  { email: "ben3@test.com", responded: false },
];

console.log(_.pick(person, ["name", "age"])); //{ name: 'ben', age: 35 }

console.log(_.uniqBy(list, "email", "responded")); // Removes one of the objects with the same 'email' and 'responded'.
// [
//   { email: "ben1@test.com", responded: false, age: 10 },
//   { email: "ben2@test.com", responded: true },
//   { email: "ben3@test.com", responded: false },
//   undefined,
// ];

console.log(_.compact(list)); // Removes undefined values from the array.
// [
//   { email: "ben1@test.com", responded: false, age: 10 },
//   { email: "ben1@test.com", responded: false, age: 20 },
//   { email: "ben2@test.com", responded: true },
//   { email: "ben3@test.com", responded: false },
// ];

console.log(_.map(list2, ({ email }) => email)); // [ 'ben1@test.com', 'ben1@test.com', 'ben2@test.com', 'ben3@test.com' ]

_.each(list2, ({ email }) => {
  console.log(email);
});
// ben1@test.com
// ben1@test.com
// ben2@test.com
// ben3@test.com
```

- We can chain all the methods that receives an array and returns an array:

```js
const _ = require("lodash");

list = [
  { email: "ben1@test.com", responded: false, age: 10 },
  { email: "ben1@test.com", responded: false, age: 20 },
  { email: "ben2@test.com", responded: true },
  { email: "ben3@test.com", responded: false },
  undefined,
];

_.chain(list)
  .compact()
  .uniqBy("email", "responded")
  .map(({ email }) => email)
  .each((email) => {
    console.log(email);
  })
  .value();

// ben1@test.com
// ben2@test.com
// ben3@test.com
```

# Payment with Stripe

```dos
npm i stripe
```

- To charge a credit card or other payment source, you create a Charge object:

```js
const express = require("express");
const config = require("config");
const stripe = require("stripe")(config.get("stripeApiKey"));
const auth = require("../middlewares/auth");
const router = express.Router();

router.post("/", auth, async (req, res) => {
  const stripeToken = req.body.id;

  await stripe.charges.create({
    amount: 500,
    currency: "usd",
    description: "$50 for 50 email credits",
    source: stripeToken,
  });

  req.user.credits += 50;
  const user = await req.user.save();

  res.send(user);
});

module.exports = router;
```

- Creare a `requireCredits.js` middleware, and use it after `auth` for the routes that requires credit:

```js
module.exports = (req, res, next) => {
  if (req.user.credits < 1) {
    return res.status(402).send({ error: "Not enough credits." });
  }

  next();
};
```

- Use `Payments` component in the front-end:

```js
import React from "react";
import StripeCheckout from "react-stripe-checkout";
import { connect } from "react-redux";
import { handleStripeToken } from "../actions/billingActions";

const Payments = ({ handleStripeToken }) => {
  return (
    <StripeCheckout
      name="Survey Sender"
      description="$50 for 50 email credits"
      amount={5000}
      token={(stripeToken) => handleStripeToken(stripeToken)}
      stripeKey={process.env.REACT_APP_STRIPE_KEY}
    >
      <button className="btn">Add Credits</button>
    </StripeCheckout>
  );
};

export default connect(null, { handleStripeToken })(Payments);
```

- and the action is:

```js
import backend from "../apis/backend";
import { FETCH_USER } from "./types";

export const handleStripeToken = (stripeToken) => async (dispatch) => {
  const { data: user } = await backend.post("/api/stripe", stripeToken);

  dispatch({ type: FETCH_USER, payload: user });
};
```

# WebSocket with socket.io

```dos
npm i socket.io
```

- Create a folder `chat` with an `index.js` in it:

```js
const io = require("socket.io")();

module.exports = function (app) {
  io.on("connection", (socket) => {
    console.log("New WebSocket connection");
  });

  app.io = io;
};
```

- Change the `app.js`:

```js
const app = require("express")();

require("./chat")(app);
//...

module.exports = app;
```

- Change the `index.js`:

```js
const app = require("./app");
const config = require("config");

var server = require("http").createServer(app);
const io = app.io;
io.attach(server);

const PORT = process.env.PORT || config.get("port");

server.listen(PORT, () => {
  console.log(`Server is up on port ${PORT}`);
});
```

## Chat example

- The `index.js` in the chat folder can be:

```js
const io = require("socket.io")();
const Filter = require("bad-words");

const { generateMessage } = require("./utils/messages");
const {
  addUser,
  removeUser,
  getUser,
  getUsersInRoom,
} = require("./utils/users");

module.exports = function (app) {
  io.on("connection", (socket) => {
    console.log("New WebSocket connection");

    socket.on("join", (options) => {
      //when the user sends a message with the "join" event.
      const { error, user } = addUser({ id: socket.id, ...options });

      if (error) return socket.emit("errorMessage", error);

      socket.join(user.room);

      socket.emit("message", generateMessage("Admin", "Welcome!"));

      socket.broadcast
        .to(user.room)
        .emit(
          "message",
          generateMessage("Admin", `${user.username} has joined!`)
        );

      io.to(user.room).emit("roomData", {
        room: user.room,
        users: getUsersInRoom(user.room),
      });
    });

    socket.on("sendMessage", (message) => {
      //when the user sends a message with the "sendMessage" event.
      const user = getUser(socket.id);
      const filter = new Filter();

      if (filter.isProfane(message))
        return socket.emit("errorMessage", "Profanity is not allowed.");

      io.to(user.room).emit("message", generateMessage(user.username, message));
    });

    socket.on("disconnect", () => {
      //when the user closes the connection.
      const user = removeUser(socket.id);

      if (user) {
        io.to(user.room).emit(
          "message",
          generateMessage("Admin", `${user.username} has left!`)
        );

        io.to(user.room).emit("roomData", {
          room: user.room,
          users: getUsersInRoom(user.room),
        });
      }
    });
  });

  app.io = io;
};
```

- In which, `/utils/messages.js`:

```js
const generateMessage = (username, text) => {
  return {
    username,
    text,
    createdAt: new Date().getTime(),
  };
};

module.exports = {
  generateMessage,
};
```

- and `/utils/users.js`:

```js
const users = [];

const addUser = ({ id, username, room }) => {
  // Clean the data
  username = username && username.trim().toLowerCase();
  room = room && room.trim().toLowerCase();

  // Validate the data
  if (!username || !room) {
    return {
      error: "Username and room are required!",
    };
  }

  // Check for existing user
  const existingUser = users.find((user) => {
    return user.room === room && user.username === username;
  });

  // Validate username
  if (existingUser) {
    return {
      error: "Username is in use!",
    };
  }

  // Store user
  const user = { id, username, room };
  users.push(user);
  return { user };
};

const removeUser = (id) => {
  const index = users.findIndex((user) => user.id === id);

  if (index !== -1) {
    return users.splice(index, 1)[0];
  }
};

const getUser = (id) => {
  return users.find((user) => user.id === id);
};

const getUsersInRoom = (room) => {
  room = room.trim().toLowerCase();
  return users.filter((user) => user.room === room);
};

module.exports = {
  addUser,
  removeUser,
  getUser,
  getUsersInRoom,
};
```

### Testing the client

- Install `chrome socket.io extension` which is like postman for `websocket`.
- `join` a user:

![](/md/chat1.jpg)

- `sendMessage` a user:

![](/md/chat2.jpg)

- When a user closes the window:

![](/md/chat3.jpg)

# Jest

```dos
npm i jest --save-dev
```

- Change test script in `package.json` to:

```json
"test": "jest --watchAll --runInBand --verbose --coverage",
```

- Create a folder `tests` in the root of the project with two sub-folders `unit` and `integration`.

## Unit Testing

- Test modules withour their external dependencies which is time consuming.
- In the unit test folder, we mimic the structure of the project and just add `.test.js` instead of each `.js` module.
- After writing our tests like below, we run `npm test` which will run jest. Note that node will not run and `test` (or its alias `it`) is a function that jest knows. So we don't import jest, we run jest.
- `--watchAll` is like nodemon.

```js
test("first test", () => {});
```

- In the unit test, first we import the things that we want to test (functions or classes) from the corresponding module (of course they should be exported). So `const { absolute } = require("../temp");`.
- In the unit test, we should write tests for each excutation paths.
- Suppose that we have a `temp.js` module:

```js
const absolute = (n) => (n >= 0 ? n : -1 * n);

const greet = (name) => `Welcome ${name}`;

const getCurrencies = () => ["USD", "AUD", "EUR"];

const getProduct = () => {
  return { id: 1, price: 10 };
};

const registerUser = (username) => {
  if (!username) throw new Error("Username is required.");

  return { id: new Date().getTime(), username };
};

module.exports = { absolute, greet, getCurrencies, getProduct, registerUser };
```

```js
const {
  absolute,
  greet,
  getCurrencies,
  getProduct,
  registerUser,
} = require("../temp");

describe("absolute", () => {
  // For grouping related tests.
  it("should return a positive number if input is positive", () => {
    const result = absolute(1);

    expect(result).toBe(1); //There are a lot of matcher functions. toBeCloseTo(0.3) for floating point numbers.
  });

  it("should return a positive number if input is negative", () => {
    const result = absolute(-1);

    expect(result).toBe(1);
  });

  it("should return 0 if input is 0", () => {
    const result = absolute(0);

    expect(result).toBe(0);
  });
});
```

### Testing strings

```js
describe("greet", () => {
  it("should return the greeting message", () => {
    const result = greet("Ben");

    expect(result).toBe("Welcome Ben"); //too specific
    expect(result).toMatch(/Ben/); //good
    expect(result).toContain("Ben"); //good
  });
});
```

### Testing arrays

```js
describe("getCurrencies", () => {
  it("should return supported currencies", () => {
    const result = getCurrencies();

    expect(result).toBeDefined(); //too general
    expect(result).not.toBeNull(); //too general

    expect(result[0]).toBe("USD"); //too specific
    expect(result[1]).toBe("AUD"); //too specific
    expect(result[2]).toBe("EUR"); //too specific
    expect(result.length).toBe(3); //too specific

    expect(result).toContain("USD"); //good

    expect(result).toEqual(expect.arrayContaining(["EUR", "USD", "AUD"])); //ideal
  });
});
```

### Testing objects

```js
describe("getProduct", () => {
  it("should return the product", () => {
    const result = getProduct();

    expect(result).toEqual({ id: 1, price: 10 }); //toBe is not working in case of objects
    expect(result).toMatchObject({ price: 10 }); //toEqual will fail here
    expect(result).toHaveProperty("price", 10); //similar to above
  });
});
```

### Testing exceptions

```js
describe("registerUser", () => {
  it("should throw if username is falsy", () => {
    //jest does not support parameterized tests
    const args = [null, undefined, 0, NaN, "", false];
    args.forEach((a) => {
      expect(() => registerUser(a)).toThrow();
    });
  });

  it("should register the user and returns it if valid username is provided", () => {
    const result = registerUser("Ben");
    expect(result).toMatchObject({ username: "Ben" });
    expect(result).toHaveProperty("id");
  });
});
```

### Mocks

- Suppose we have a `temp.js` which uses two external resources `db` and `mail`:

```js
const db = require("./db");
const mail = require("./mail");

const applyDiscount = async (order) => {
  const customer = await db.getCustomer(order.customerId);

  if (customer.points > 10) {
    order.totalPrice *= 0.9;
  }
};

const notifyCustomer = async (order) => {
  const customer = await db.getCustomer(order.customerId);

  mail.send(customer.email, "Your order was placed.");
};

module.exports = { applyDiscount, notifyCustomer };
```

- `db.js`:

```js
const getCustomer = (id) => {
  console.log("Reading from db...");
  return Promise.resolve({ id, email: "customer@test.com", points: 15 });
};

module.exports = {
  getCustomer,
};
```

- `mail.js`:

```js
const send = (email, body) => {
  console.log(`Sending ${body} to ${email}`);
};

module.exports = {
  send,
};
```

- Now if we want to test `temp.js`, in `temp.test.js`, we have problems:

```js
const { applyDiscount, notifyCustomer } = require("../temp");

describe("applyDiscount", () => {
  it("should apply 10% discount when customer has more than 10 points", async () => {
    const order = { customerId: 2, totalPrice: 100 };
    await applyDiscount(order); //It is using the real db.

    expect(order.totalPrice).toBe(90);
  });
});

describe("notifyCustomer", () => {
  it("should send an email", async () => {
    await notifyCustomer({ customerId: 2 }); // It using the real db and email service and we can't tell if the send method is called or not.
  });
});
```

- To solve the problem, we have to import `db` and `mail` modules and change them (you cannot change the functions if you desctructure them, because they are constants) to the mocks in the test file:

```js
const db = require("../db");
const mail = require("../mail");
const { applyDiscount, notifyCustomer } = require("../temp");

describe("applyDiscount", () => {
  it("should apply 10% discount when customer has more than 10 points", async () => {
    db.getCustomer = jest.fn().mockResolvedValue({ points: 15 }); //if the function returns a promise (resolved)
    // jest.fn().mockReturnValue(1); //if the function does not return a promise
    // jest.fn().mockRejectedValue(new Error('...')); //if the function returns a promise (rejected)

    const order = { customerId: 2, totalPrice: 100 };
    await applyDiscount(order); //Now, it is using the mock db.getCustomer

    expect(order.totalPrice).toBe(90);
  });
});

describe("notifyCustomer", () => {
  it("should send an email", async () => {
    db.getCustomer = jest.fn().mockResolvedValue({ email: "a" });
    mail.send = jest.fn();

    await notifyCustomer({ customerId: 2 }); //Now, it is using the mock db.getCustomer and mail.send

    expect(mail.send).toHaveBeenCalled();
    expect(mail.send).toHaveBeenCalledWith("a", "Your order was placed."); //too specific
  });
});
```

## Integration Testing

- In integration tests, sometimes we want to integrate the db but mock the email service.
- For integration testing,
  1. Populate the test db
  2. Send Http requests
  3. Assert
- In the `config` folder and in `test.json`:

```json
{
  "db": "mongodb+srv://admin:Pa$$w0rd@sample-f7ktv.mongodb.net/test?retryWrites=true&w=majority"
}
```

- Install `supertest` which is like `postman`:

```dos
npm i supertest --save-dev
```

- Create a folder called `fixtures` in the `integration` folder and create a `db.js` file in it:

```js
const mongoose = require("mongoose");
const { Value } = require("../../../models/value");

const valueOne = {
  _id: new mongoose.Types.ObjectId(),
  amount: 100,
};

const valueTwo = {
  _id: new mongoose.Types.ObjectId(),
  amount: 200,
};

const valueThree = {
  _id: new mongoose.Types.ObjectId(),
  amount: 300,
};

const setupDatabase = async () => {
  await Value.deleteMany();
  await new Value(valueOne).save();
  await new Value(valueTwo).save();
  await new Value(valueThree).save();
};

module.exports = {
  valueOne,
  setupDatabase,
};
```

- Create a `values.test.js` in the `integration` folder:

```js
const request = require("supertest");
const mongoose = require("mongoose");
const app = require("../../app");
const { Value } = require("../../models/value");
const { valueOne, setupDatabase } = require("./fixtures/db");

describe("/api/values", () => {
  beforeEach(async () => {
    await setupDatabase();
  });

  describe("GET /", () => {
    it("should return all values", async () => {
      const res = await request(app).get("/api/values");

      expect(res.status).toBe(200);
      expect(res.body.length).toBe(3);
      expect(res.body.some((v) => v.amount === 100)).toBeTruthy();
      expect(res.body.some((v) => v.amount === 200)).toBeTruthy();
      expect(res.body.some((v) => v.amount === 300)).toBeTruthy();
    });
  });

  describe("GET /:id", () => {
    it("should return a value if a valid id is passed.", async () => {
      const res = await request(app).get(`/api/values/${valueOne._id}`);

      expect(res.status).toBe(200);
      expect(res.body).toHaveProperty("amount", valueOne.amount);
    });

    it("should return 404 if no value with the given id exists.", async () => {
      const res = await request(app).get(
        `/api/values/${mongoose.Types.ObjectId()}`
      );

      expect(res.status).toBe(404);
    });
  });

  describe("POST /", () => {
    // Define the happy path, and then in each test, we change
    // one parameter that clearly aligns with the name of the
    // test.
    const exec = async () => {
      return (res = await request(app).post("/api/values/").send({ amount }));
    };

    beforeEach(() => {
      amount = 400;
    });

    it("should return 400 if value is larger than 1000", async () => {
      amount = 1001;

      const res = await exec();

      expect(res.status).toBe(400);
    });

    it("should save the value if it is valid", async () => {
      await exec();

      const value = await Value.find({ amount: 400 });

      expect(value).not.toBeNull();
    });

    it("should return the value if it is valid", async () => {
      const res = await exec();

      expect(res.body).toHaveProperty("_id");
      expect(res.body).toHaveProperty("amount", 400);
    });
  });

  describe("PUT /:id", () => {
    const exec = async () => {
      return await request(app)
        .put(`/api/values/${id}`)
        .send({ amount: newAmount });
    };

    beforeEach(async () => {
      id = valueOne._id;
      newAmount = 400;
    });

    it("should return 400 if value is larger than 1000", async () => {
      newAmount = 1001;

      const res = await exec();

      expect(res.status).toBe(400);
    });

    it("should return 404 if value with the given id was not found", async () => {
      id = mongoose.Types.ObjectId();

      const res = await exec();

      expect(res.status).toBe(404);
    });

    it("should update the value if input is valid", async () => {
      await exec();

      const updatedValue = await Value.findById(valueOne._id);

      expect(updatedValue.amount).toBe(newAmount);
    });

    it("should return the updated value if it is valid", async () => {
      const res = await exec();

      expect(res.body).toHaveProperty("_id");
      expect(res.body).toHaveProperty("amount", newAmount);
    });
  });

  describe("DELETE /:id", () => {
    const exec = async () => {
      return await request(app).delete(`/api/values/${id}`).send();
    };

    beforeEach(async () => {
      id = valueOne._id;
    });

    it("should return 404 if no value with the given id was found", async () => {
      id = mongoose.Types.ObjectId();

      const res = await exec();

      expect(res.status).toBe(404);
    });

    it("should delete the value if input is valid", async () => {
      await exec();

      const valueInDb = await Value.findById(id);

      expect(valueInDb).toBeNull();
    });

    it("should return the removed value", async () => {
      const res = await exec();

      expect(res.body).toHaveProperty("_id", valueOne._id.toHexString());
      expect(res.body).toHaveProperty("amount", valueOne.amount);
    });
  });
});
```

# Security

## Denial-Of-Service (DOS) Attacks

### Solution

- Limit the body payload using body-parser.

```js
app.use(express.json({ limit: "10kb" }));
```

- Use `express-rate-limit`:

```js
const rateLimit = require("express-rate-limit");

// Enable if you're behind a reverse proxy (Heroku, Bluemix, AWS ELB, Nginx, etc)
// see https://expressjs.com/en/guide/behind-proxies.html
// app.set('trust proxy', 1); // It means that trust only the first hop

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
});
app.use("/api/", apiLimiter);

const createAccountLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour window
  max: 5, // start blocking after 5 requests
  message:
    "Too many accounts created from this IP, please try again after an hour",
});
app.post("/create-account", createAccountLimiter, function (req, res) {
  //...
});
```

## Cross-Site Scripting (XSS) Attacks

- Attacker injects malicious JavaScript into the victim's document by tricking a victim into clicking on a link to a legit website but with some script along with it to be run as you (reflected -> impacts just the currnt user) or injetcs malicious JS to the server side by entering script in a form input (stored attcks -> impacts everybody).
- XSS occurs on input forms when those forms are not validated or encoded.
- In XSS, attacker can gain access to cookies, session tokens and/or other sensitive data. As well, these scripts can rewrite HTML content of a page.
- These attacks happen in server-side-rendered apps -> not in React.

### Solution

- Sanitize your input.
- Do not write inline script in a html. Always write it in a separate JS file and tell the client (browser) that you trust this JS file with this header in the response (for the routes that sends html):

```js
res.setHeader(("Content-Security-Policy": "script-src 'none'")); // means this html has no script
res.setHeader(("Content-Security-Policy": "script-src http://localhost:8080")); // or, only trust this source
```

- Use `helmet`.
- Use React.

## Cross-Site Request Forgery (CSRF) Attacks

- Attacker tricks the client to click on a link to make an unwanted request to the server.
- XSS is more dangerous than CSRF because it is a two-way attack and CSRF is one-way.

### Solution

- Use CSRF token. A CSRF token is a unique, secret, unpredictable value that is generated by the server-side application and transmitted to the client in such a way that it is included in a subsequent HTTP request made by the client.

## Server-Side Request Forgery (SSRF) Attacks

- It is when our API talks to a 3rd-party API and receives the url to that 3rd-party API from the client! The client can send a fake url and makes our API to make an unwanted call.

### Solution

- Never trust the client.

## Brute Force Attacks

- Brute force attack is a method used to obtain sensitive data such as user password by sending too many requests of gusses to auth routes.

### Solution

- Using rate-limiters for auth routes.
- Two-factor authentication

## SQL/NoSQL Injection Attacks

- This attack happens when you build your query based on user input.

### Solution

- Use ORM/ODM (`sequelize` or `mongoose`).
- Use parameterized queries or prepared statements.

# Deployment

```dos
npm i helmet compression
```

- In the `startup` folder, add a new module `prod.js`:

```js
const const helmet = require("helmet");
const compression = require("compression");

module.exports = function (app) {
  app.use(helmet());
  app.use(compression());
};
```

- In the `app.js`, add the following after all `require` statements:

```js
if (app.get("env") === "production") {
  require("./startup/prod")(app);
}
```

- In `package.json`, add the following under `scripts` section:

```json
"start": "node index.js"
```

- In `package.json`, add the following in root:

```json
"engines": {
  "node": "9.2.3"
},
```

# Gulp task runner

```dos
 npm i -g gulp
 npm init -y
 npm i gulp --save-dev
```

- Create a `gulpfile.js` in the root to describe all the tasks in it:

```js
const gulp = require("gulp");

gulp.task("message", async () => {
  console.log("Gulp1 is running...");
});

gulp.task("default", async () => {
  console.log("Gulp2 is running...");
});
```

- Now by running:

```dos
gulp message

[21:29:55] Starting 'message'...
Gulp1 is running...
[21:29:55] Finished 'message' after 5.15 ms
```

- And by running:

```dos
gulp

[21:30:17] Starting 'default'...
Gulp2 is running...
[21:30:17] Finished 'default' after 4.93 ms
```

## Gulp functions

- `gulp.task` -> Define tasks.
- `gulp.src` -> Point to files to use.
- `gulp.dest` -> Points to folder to output.
- `gulp.watch` -> Watch files and folders for changes.

## Create a task to copy two files from `src` to `dist`

- Create a `src` folder in the root.
- Put two html files (index and about) in it.
- Add the following in the `gulpfile.js`:

```js
// Copy All HTML files
gulp.task("copyHtml", async () => {
  gulp.src("src/*.html").pipe(gulp.dest("dist"));
});
```

```dos
gulp copyHtml

[21:37:32] Starting 'copyHtml'...
[21:37:32] Finished 'copyHtml' after 16 ms
```

- It creates a `dist` folder and copies the two files in the `src` folder in it.

## Gulp plugins

### gulp-imagemin

```dos
npm i gulp-imagemin --save-dev
```

- It optimize images automatically.
- Create a folder `images` in the `src` folder and put some images in it.

```js
const imagemin = require("gulp-imagemin");

// Optimize Images
gulp.task("imageMin", async () =>
  gulp.src("src/images/*").pipe(imagemin()).pipe(gulp.dest("dist/images"))
);
```

```dos
gulp imageMin

[22:02:04] Starting 'imageMin'...
[22:02:04] Finished 'imageMin' after 15 ms
[22:02:07] gulp-imagemin: Minified 2 images (saved 4.65 MB - 67.2%)
```

### gulp-uglify

```dos
npm i gulp-uglify --save-dev
```

- It removes white space in files.
- Create a folder `js` in the `src` folder and put a js file in it with white spaces between lines in it.

```js
const uglify = require("gulp-uglify");

// Minify JS
gulp.task("minify", async () => {
  gulp.src("src/js/*.js").pipe(uglify()).pipe(gulp.dest("dist/js"));
});
```

```dos
gulp minify

[22:05:50] Starting 'minify'...
[22:05:50] Finished 'minify' after 17 ms
```

### gulp-concat

```dos
npm i gulp-concat --save-dev
```

- It merge files together.

```js
const concat = require("gulp-concat");
const uglify = require("gulp-uglify");

// Scripts
gulp.task("scripts", async () => {
  gulp
    .src("src/js/*.js")
    .pipe(concat("main.js"))
    .pipe(uglify())
    .pipe(gulp.dest("dist/js"));
});
```

```dos
gulp scripts

[22:08:21] Starting 'scripts'...
[22:08:21] Finished 'scripts' after 17 ms
```

- It merges all files in the `src/js` folder to the `main.js` file in the `dist/js` folder.

## Multiple gulp tasks

- Instead of one task in the `default`, we can have multiple tasks inside an array:

```js
gulp.task(
  "default",
  gulp.series(["message", "copyHtml", "imageMin", "scripts"])
);
```

```dos
gulp

[22:21:27] Using gulpfile ~\Desktop\js\gulpfile.js
[22:21:27] Starting 'default'...
[22:21:27] Starting 'message'...
Gulp1 is running...
[22:21:27] Finished 'message' after 3.56 ms
[22:21:27] Starting 'copyHtml'...
[22:21:27] Finished 'copyHtml' after 14 ms
[22:21:27] Starting 'imageMin'...
[22:21:27] Finished 'imageMin' after 4.23 ms
[22:21:27] Starting 'scripts'...
[22:21:27] Finished 'scripts' after 3.76 ms
[22:21:27] Finished 'default' after 38 ms
[22:21:30] gulp-imagemin: Minified 2 images (saved 4.65 MB - 67.2%)
```

## Watch

- It is like running the related task (in the second parameter), when the files (in the first parameter) change.

```js
gulp.task("watch", async () => {
  gulp.watch("src/js/*.js", gulp.series(["scripts"]));
  gulp.watch("src/images/*", gulp.series(["imageMin"]));
  gulp.watch("src/*.html", gulp.series(["copyHtml"]));
});
```

```dos
gulp watch

```

# Absolute imports

- You can configure your application to support importing modules using absolute paths. This can be done by configuring a `jsconfig.json` or `tsconfig.json` file in the root of your project. If you're using TypeScript in your project, you will already have a tsconfig.json file.

```json
{
  "compilerOptions": {
    "baseUrl": "src"
  },
  "include": ["src"]
}
```

# Cross Origin Resource Sharing

- A security feature implemented in `browsers`.
- We should configure the server to allow CORS.
- `npm i cors`.
- Then in `startup/cors.js`:

```js
import cors from "cors";

export default function (app) {
  app.use(cors());
}
```

- Then in `app.js`:

```js
const app = express();
cors(app);
```
