# MicroFront-ends

![](/md/145.jpg)

## Requirements

- Because if we want to replace an app, we don't want to make changes to other apps that were entangled with the app that we want to replace.

![](/md/146.jpg)

- The top-bar is rendered by the container, so there will be a tiny communication between child/parent.

![](/md/147.jpg)

- The CSS should be scoped.

![](/md/148.jpg)

![](/md/149.jpg)

![](/md/150.jpg)

## Linking Multiple Apps together

- We are not using create-react-app, because the version of the webpack in it does not support module federation plugin (this plugin exposes a remoteEntry.js file for each child app which contains a list of files that are available from this child project + directions on how to load them. This plugin is used to integrate child and container apps).
- For example in `container` app, it should be something like this:

![](/md/151.jpg)

- And in `child` project:

![](/md/152.jpg)

- Folder structure for each project:

![](/md/153.jpg)

### Changes in the child app

- For each project, we have to create three webpack config files: common, only for dev, only for prod. So create a `config` folder in each project:
- `config/webpack.common.js`:

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.m?js$/, // run all js and mjs files through babel
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-react", "@babel/preset-env"], // first preset is for jsx
            plugins: ["@babel/plugin-transform-runtime"],
          },
        },
      },
    ],
  },
};
```

- `config/webpack.dev.js`:

```js
const { merge } = require("webpack-merge");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const commonConfig = require("./webpack.common");
const packageJson = require("../package.json");

const devConfig = {
  mode: "development",
  output: {
    publicPath: "http://localhost:8081/", // if we have nested routes in the child app we have to add this so we add this any way
  },
  devServer: {
    port: 8081, // it will serve the html in this port
    historyApiFallback: {
      index: "index.html",
    },
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "marketing",
      filename: "remoteEntry.js",
      exposes: {
        "./MarketingApp": "./src/bootstrap",
      },
      shared: packageJson.dependencies, // we don't want to load dependencies multiple times in the container app
    }),
    new HtmlWebpackPlugin({
      // this one add script tag automatically to this html
      template: "./public/index.html",
    }),
  ],
};

module.exports = merge(commonConfig, devConfig); // commonConfig should come first to be able to overridden
```

- `config/webpack.prod.js`:

```js
const { merge } = require("webpack-merge");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const packageJson = require("../package.json");
const commonConfig = require("./webpack.common");

const prodConfig = {
  mode: "production", // it minifies the output and some other optimizations
  output: {
    filename: "[name].[contenthash].js", // name of the file and then hash of its content to defeat caching problems
    publicPath: "/marketing/latest/", // so the ModuleFederationPlugin prepends the address to the urls in the remoteEntry file that is going to generate
  },
  plugins: [
    // in prod, we don't need to use HtmlWebpackPlugin because there will be no html from this child app
    new ModuleFederationPlugin({
      // although this plugin is the same as the dev version, we don't extract them in common module for future scenarios
      name: "marketing",
      filename: "remoteEntry.js",
      exposes: {
        "./MarketingApp": "./src/bootstrap",
      },
      shared: packageJson.dependencies,
    }),
  ],
};

module.exports = merge(commonConfig, prodConfig);
```

- For each project, we create a `public` folder as well with `public/index.html` in it:

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <div id="_marketing-dev-root"></div>
    <!-- An id which is unlikely for our container app to have a similar one  -->
  </body>
</html>
```

- `src/index.js` is just to import `src/bootstrap.js` which gives some time to the browser to load other modules:

```js
import("./bootstrap"); // it is import function not statement
```

- `src/bootstrap.js` is the main logic:

```js
import React from "react";
import ReactDOM from "react-dom";
import { createMemoryHistory, createBrowserHistory } from "history";
import App from "./App";

// Mount function to start up the app
const mount = (el, { onNavigate, defaultHistory, initialPath }) => {
  const history =
    defaultHistory ||
    createMemoryHistory({
      initialEntries: [initialPath], // because memory history always starts at "/" but we don't want that
    }); // use Browser history when running child app in isolation

  if (onNavigate) {
    history.listen(onNavigate); // whenever navigation occurs, history object is going to call whatever passed in the listen method.
  }

  ReactDOM.render(<App history={history} />, el);

  return {
    onParentNavigate({ pathname: nextPathname }) {
      const { pathname } = history.location;

      if (pathname !== nextPathname) {
        // to prevent infinite loop
        history.push(nextPathname);
      }
    },
  };
};

// If we are in development and in isolation,
// call mount immediately
if (process.env.NODE_ENV === "development") {
  // it is set by webpack
  const devRoot = document.querySelector("#_marketing-dev-root");

  if (devRoot) {
    mount(devRoot, { defaultHistory: createBrowserHistory() });
  }
}

// We are running through container
// and we should export the mount function
export { mount };
```

- Also in `./package.json`:

```json
"scripts": {
    "start": "webpack serve --config config/webpack.dev.js",
    "build": "webpack --config config/webpack.prod.js" // we are not serving anything in production, we only want to build it
  },
```

- `src/App.js`:

```js
import React from "react";
import { Switch, Route, Router } from "react-router-dom";
import { StylesProvider } from "@material-ui/core/styles";

import Landing from "./components/Landing";
import Pricing from "./components/Pricing";

export default ({ history }) => {
  return (
    <div>
      <StylesProvider>
        <Router history={history}>
          <Switch>
            <Route exact path="/pricing" component={Pricing} />
            <Route path="/" component={Landing} />
          </Switch>
        </Router>
      </StylesProvider>
    </div>
  );
};
```

### Changes in the container app

- The `config` folder is similar to the child app's (there are the exact same three files and also the change in `package.json` file).
  The `config/webpack.common.js`:

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-react", "@babel/preset-env"],
            plugins: ["@babel/plugin-transform-runtime"],
          },
        },
      },
    ],
  },
  plugins: [
    // because for container app, we will need html output both for dev and prod
    new HtmlWebpackPlugin({
      template: "./public/index.html",
    }),
  ],
};
```

The `config/webpack.dev.js`:

```js
const { merge } = require("webpack-merge");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const commonConfig = require("./webpack.common");
const packageJson = require("../package.json");

const devConfig = {
  mode: "development",
  output: {
    publicPath: "http://localhost:8080/",
  },
  devServer: {
    port: 8080, // a different port from child app
    historyApiFallback: {
      index: "index.html",
    },
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "container",
      remotes: {
        marketing: "marketing@http://localhost:8081/remoteEntry.js", // the first marketing should match with -> import { mount } from "marketing/MarketingApp"; <- and the second marketing should match with the name in webpack.dev.js of child app.
        auth: "auth@http://localhost:8082/remoteEntry.js",
        dashboard: "dashboard@http://localhost:8083/remoteEntry.js",
      },
      shared: packageJson.dependencies,
    }),
  ],
};

module.exports = merge(commonConfig, devConfig);
```

For `config/webpack.prod.js`:

```js
const { merge } = require("webpack-merge");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const commonConfig = require("./webpack.common");
const packageJson = require("../package.json");

const domain = process.env.PRODUCTION_DOMAIN; // this needs to be exposed in the build env

const prodConfig = {
  mode: "production",
  output: {
    filename: "[name].[contenthash].js",
    publicPath: "/container/latest/", // so the HtmlWebpackPlugin prepends the address to main.asdsad.js file with this option
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "container",
      remotes: {
        marketing: `marketing@${domain}/marketing/latest/remoteEntry.js`, // so the marketing is a folder for marketing child app
        auth: `auth@${domain}/auth/latest/remoteEntry.js`,
        dashboard: `dashboard@${domain}/dashboard/latest/remoteEntry.js`,
      },
      shared: packageJson.dependencies,
    }),
  ],
};

module.exports = merge(commonConfig, prodConfig);
```

- In `public` folder, the `public/index.html`:

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

- In the `src` folder, `src/index.js`:

```js
import("./bootstrap");
```

and `src/bootstrap.js`, we don't need to define any `mount` function, because we want to display things right away:

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(<App />, document.querySelector("#root"));
```

and `src/App.js`:

```js
import React, { lazy, Suspense, useState, useEffect } from "react"; // lazy is a function and Suspense is a component. We use them to lazy load the components so we don't get all the js code for all child apps at the start of the container.
import { Router, Route, Switch, Redirect } from "react-router-dom";
import { createBrowserHistory } from "history";

import Header from "./components/Header";
const AuthLazy = lazy(() => import("./components/AuthApp"));
const MarketingLazy = lazy(() => import("./components/MarketingApp"));
const DashboardLazy = lazy(() => import("./components/DashboardApp"));

const history = createBrowserHistory();

export default () => {
  const [isSignedIn, setIsSignedIn] = useState(false);

  useEffect(() => {
    if (isSignedIn) {
      history.push("/dashboard");
    }
  }, [isSignedIn]);

  return (
    <Router history={history}>
      <div>
        <Header onSignOut={() => setIsSignedIn(false)} isSignedIn={isSignedIn} />
        <Suspense fallback={<div>loading...</div>}>
          <Switch>
            <Route path="/auth">
              <AuthLazy onSignIn={() => setIsSignedIn(true)} />
            </Route>
            <Route path="/dashboard">
              {!isSignedIn && <Redirect to="/" />}
              <DashboardLazy />
            </Route>
            <Route path="/" component={MarketingLazy} />
          </Switch>
        </Suspense>
      </div>
    </Router>
  );
};
```

and `src/components/MarketingApp.js`:

```js
import { mount } from "marketing/MarketingApp";
import React, { useRef, useEffect } from "react";
import { useHistory } from "react-router-dom";

export default () => {
  const ref = useRef(null);
  const history = useHistory();

  useEffect(() => {
    const { onParentNavigate } = mount(ref.current, {
      initialPath: history.location.pathname,
      onNavigate: ({ pathname: nextPathname }) => {
        // the listen method of the history object in the child app, calls the callback with a location object.
        const { pathname } = history.location;

        if (pathname !== nextPathname) {
          // to prevent infinite loop
          history.push(nextPathname);
        }
      },
    }); // it runs after every render. If we haven't used useEffect, it would run first and then render which makes no sense.

    history.listen(onParentNavigate);
  }, []);

  return <div ref={ref} />;
};
```

- The reason that we didn't export a React component from child app and import in the container app is to have almost zero coupling between these two and have no assumption about the framework that the child has used.

## CI/CD

![](/md/154.jpg)

- We want to have `monorepo` in Github. When each project folder has changed, we want to build a production version of that project (which has a main.dfg2125421.js (bundle from webpack), dependency js files and index.html for the container app and for child projects, it has a main.25137454c.js + remoteEntry.js + dependency files) and upload them to Amazon S3. Then `Amazon CloudFront` which is a CDN will serve these files to the browser: index.html from container -> main.js from container -> remoteEntry.js from child -> main.js from child...

- We will use Github actions which are snippets of codes to be run when an event (e.g. push code) occurs.

### CI/CD for Container app

![](/md/155.jpg)

- To do that, create a `.github` folder in the root of the project, and inside that create a `workflows` sub-folder. All the files in this folder will be picked-up by Github. Inside that `container.yml`:

```yml
name: deploy-container

on:
  push: # push event
    branches: # only on master -> only one entry
      - master
    paths: # changes only in the following folder
      - "packages/container/**"

defaults:
  run:
    working-directory: packages/container # this section sets the working directory for the following commands

jobs:
  build: # we want one job to build and deploy the app
    runs-on: ubuntu-latest # a virtual machine runs by Github

    steps:
      - uses: actions/checkout@v2 # checkout the code and load it in the virtual machine
      - run: npm install
      - run: npm run build # it creates a dist directory
        env:
          PRODUCTION_DOMAIN: ${{ secrets.PRODUCTION_DOMAIN }}

      - uses: chrislennon/action-aws-cli@v1.1 # somehow install aws-cli in the virtual machine
      - run: aws s3 sync dist s3://${{ secrets.AWS_S3_BUCKET_NAME }}/container/latest # dist is the output folder for webpack build
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      - run: aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_DISTRIBUTION_ID}} --paths "/container/latest/index.html" # This line has been added for manual invalidation explained below.
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

- Go to Amazon S3, create a bucket and then enable static website hosting for it in `Properties` tab (with an index document of `index.html`). Then in `Permissions` tab, un-check `Block all public access` to make it available for everyone. Actually we are not going to use this bucket directly and we will use `CloudFront` to take the files and make it available to the public. To make S3 available for CF, again in `Permissions` tab, generate a `S3 Bucket Policy` with `*` principal, `Allow` effect, `GetObject` action with the ARN of the bucket + `/*` and then paste it on the policy and save the changes.

![](/md/156.jpg)

- We will create a `distribution` (a `Web` one not an `RTMP` one) on the `CloudFront` console to pull out files from S3 and distribute it:
  - `Origin Domain Name`: -> select the bucket that we create.
  - Select `Redirect HTTP to HTTPS` instead of `HTTP and HTTPS`.
- After the distribution is created,

  - click on it and in the `General` tab, in `Default Root Object` put `/container/latest/index.html`.
  - The on `Error Pages` tab, `Create Custom Error Response` (instead of giving error, give this html to the person with 200 code):

    ![](/md/157.jpg)

  - Now you can access the CF with the domain name in the first tab.

- Now, we have to create `Access key` and `Secret access key` in AWS (by adding a user in `IAM` which only has a programmatic access and attach a policy which has s3 and CloudFront access) and come back to Github to set the secrets (in the Settings section of our repo -> Secrets). Also, we need to create `PRODUCTION_DOMAIN` with the value of `https://domain-name-of-the-cloudfront` in Github secrets and expose it in env section of the above yml file.
- For JS files, because we used hashes in their names, CF will pick them; but for our `index.html`, when we change the js in it, it will be modified but will be cached by CF so we have to invalidate the old version by going to `Invalidations` tab in `CloudFront` distributions and clicking on the `Create Invalidation` -> We took care of this manual work in the Github workflow.

### CI/CD for Marketing app (child app)

- Create `.github/workflows/marketing.yml`:

```yml
name: deploy-marketing

on:
  push:
    branches:
      - master
    paths:
      - "packages/marketing/**"

defaults:
  run:
    working-directory: packages/marketing

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm run build

      - uses: chrislennon/action-aws-cli@v1.1
      - run: aws s3 sync dist s3://${{ secrets.AWS_S3_BUCKET_NAME }}/marketing/latest
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      - run: aws cloudfront create-invalidation --distribution-id ${{ secrets.AWS_DISTRIBUTION_ID}} --paths "/marketing/latest/remoteEntry.js"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## CSS Issues

- One major issue of micro-frontend is that when navigating between different pages (apps), we might have conflicting CSS rules -> solution is to use CSS-in-JS libraries to scope the CSS and also for third-party CSS libraries (use their JS modules). Also note that if we use same CSS-in-JS library for two or more projects, they can come up with identical class names which collide with each other -> solution is to use some configuration on their side to prefix generated class names in libraries like `Material UI React component` library.

## Navigation

![](/md/158.jpg)

- Solution: For this we can add `React-Router` library to the parent and sub-apps.

![](/md/159.jpg)

- Solution: For this we can have two levels of routing logic -> in the container app, for `/` and `/pricing` we show `Marketing` app and in `Marketing` based on it, we will show `Landing` or `Pricing` pages.

![](/md/160.jpg)

- Solution: In the container, we show two different sub-apps at the same time.

![](/md/161.jpg)

![](/md/162.jpg)

![](/md/163.jpg)

- For the last three challenges, we use `Browser History` (which looks at the url in the browser after the domain name and can update it as well) in the container app and `Memory History` in the child apps.
- So instead of `BrowserHistory` we just import `Router` so we should give it our own version of history object.

![](/md/164.jpg)

- To enable this communication, we don't share history objects (because we might use different versions of react-router or even different frameworks). So we keep it as general as possible. To do that:
  - We pass `onNavigate` callback down to the child apps when calling `mount` function and whenever we navigate in a child app, we will notify the container app by call that onNavigate.

## Authentication

![](/md/165.jpg)

- We are going with option 2. In practice, we will use `currentUser` object instead of a `isSignedIn` boolean but it is fine for now.

![](/md/166.jpg)

- First, in the `AuthApp` component in the container app, we will receive `onSignIn` prop from the container app and we will pass it as a property of the second argument:

```js
import { mount } from "marketing/AuthApp";
import React, { useRef, useEffect } from "react";
import { useHistory } from "react-router-dom";

export default ({ onSignIn }) => {
  const ref = useRef(null);
  const history = useHistory();

  useEffect(() => {
    const { onParentNavigate } = mount(ref.current, {
      initialPath: history.location.pathname,
      onNavigate: ({ pathname: nextPathname }) => {
        const { pathname } = history.location;

        if (pathname !== nextPathname) {
          history.push(nextPathname);
        }
      },
      onSignIn: () => {
        onSignIn();
      },
    });

    history.listen(onParentNavigate);
  }, []);

  return <div ref={ref} />;
};
```

- We will de-structure `onSignIn` in the `mount` function (inside `bootstrap`) of the `Auth` app and pass it down to `App` component: `ReactDOM.render(<App onSignIn={onSignIn} history={history} />, el);`. Then, we will pass down and receive it in the component that handles sign in.

## Adding a Vue app (dashboard)

- In the container app `src/components/DashboardApp.js`:

```js
import { mount } from "dashboard/DashboardApp";
import React, { useRef, useEffect } from "react";

export default () => {
  const ref = useRef(null);

  useEffect(() => {
    mount(ref.current);
  }, []);

  return <div ref={ref} />;
};
```

- `config/webpack.common.js`:

```js
const { VueLoaderPlugin } = require("vue-loader");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "[name].[contenthash].js",
  },
  resolve: {
    extensions: [".js", ".vue"],
  },
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|woff|svg|eot|ttf)$/i,
        use: [{ loader: "file-loader" }],
      },
      {
        test: /\.vue$/,
        use: "vue-loader",
      },
      {
        test: /\.scss|\.css$/,
        use: ["vue-style-loader", "style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
            plugins: ["@babel/plugin-transform-runtime"],
          },
        },
      },
    ],
  },
  plugins: [new VueLoaderPlugin()],
};
```

- `config/webpack.dev.js`:

```js
const { merge } = require("webpack-merge");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const commonConfig = require("./webpack.common");
const packageJson = require("../package.json");

const devConfig = {
  mode: "development",
  output: {
    publicPath: "http://localhost:8083/",
  },
  devServer: {
    port: 8083,
    historyApiFallback: {
      index: "index.html",
    },
    headers: {
      "Access-Control-Allow-Origin": "*",
    },
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "dashboard",
      filename: "remoteEntry.js",
      exposes: {
        "./DashboardApp": "./src/bootstrap",
      },
      shared: packageJson.dependencies,
    }),
    new HtmlWebpackPlugin({
      template: "./public/index.html",
    }),
  ],
};

module.exports = merge(commonConfig, devConfig);
```

- `config/webpack.prod.js`:

```js
const { merge } = require("webpack-merge");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const packageJson = require("../package.json");
const commonConfig = require("./webpack.common");

const prodConfig = {
  mode: "production",
  output: {
    filename: "[name].[contenthash].js",
    publicPath: "/dashboard/latest/",
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "dashboard",
      filename: "remoteEntry.js",
      exposes: {
        "./DashboardApp": "./src/bootstrap",
      },
      shared: packageJson.dependencies,
    }),
  ],
};

module.exports = merge(commonConfig, prodConfig);
```

- `public/index.html`:

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <div id="_dashboard-dev-root"></div>
  </body>
</html>
```

- `src/index.js`:

```js
import("./bootstrap");
```

- `src/bootstrap.js`:

```js
import { createApp } from "vue";
import Dashboard from "./components/Dashboard.vue";

const mount = (el) => {
  const app = createApp(Dashboard);
  app.mount(el);
};

if (process.env.NODE_ENV === "development") {
  const devRoot = document.querySelector("#_dashboard-dev-root");

  if (devRoot) {
    mount(devRoot);
  }
}

export { mount };
```

- `./package.json`:

```json
"scripts": {
    "start": "webpack serve --config config/webpack.dev.js",
    "build": "webpack --config config/webpack.prod.js"
  },
```
