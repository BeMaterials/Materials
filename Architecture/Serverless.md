By adding `Access Key Id` and `Secret Access Key` for the user that you have created with the admin access in your environment variables, you will be logged in in the AWS CLI automatically.

![](/md/227.jpg)

![](/md/228.jpg)

![](/md/229.jpg)

# API Gateway

![](/md/230.jpg)

- You can create a RESTful API with it. You can also hide an API behind it.

- There are some tabs in `API Gateway`: `APIs`, `Usage Plans`, `API Keys`, `Custom Domain Names`, `Client Certificates`, and `Settings`.

  - If your API is public, you can define `API Keys` for it and track other developers' usage.
  - `Client Certificate` tab is when your API forwards the request to another HTTP endpoint and you want to authenticate your API.
  - `APIs` is what we are interested in.

- To create an API:

  - To create an API, give it a name and create it. You have the option to create an API from a `swagger` file or cloning an existing API.
  - Then, on `Actions` button, you can create a `Resource` (which will be added to URL path) by giving it a name.
  - Then, you can create a `Method` on that resource. Specify the HTTP request type (GET, POST, ...). Then you have to specify the `integration` type:
    1. `Lambda Function`
    2. `HTTP`: to forward to another HTTP endpoint.
    3. `AWS Service`: to trigger an AWS service.
    4. `Mock`: to return a dummy response, you should click on `integration response` and specify the JSON body.
  - After that, you `deploy` your API (by clicking `Deploy API` on `Actions` button) to a `stage`. For the first time, you don't have any stage, so you should create one (something like `dev`, `prod`, ...).
  - You will be given a URL.

- `APIs` tab:

  - Under `Resources`, we manage our resources and methods. Our changes are not live until we deploy them.
  - Under `Stages`, we see our deployments (a snapshot of our API) and their URLs. You can't edit here.
  - Under `Authorizers`, to protect some resources or methods, you can define authorizers here to authenticate users.
  - Under `Models`, you define the shape of the data you are working with in your API by using `JSON schema`. It's optional. You can add data validation with it.
    - You create a model (name it like MyData) and pass a schema:
    ```json
    {
      "$schema": "http://json-schema.org/draft-04/schema#",
      "title": "MyData",
      "type": "object",
      "properties": {
        "age": { "type": "integer" }
      },
      "required": ["age"]
    }
    ```
  - Under `Documentation`, you define which data to send and which data to expect as a response.
  - `Binary Support` is for registering endpoints that expect receiving files along with the request (If you don't specify the filetypes, you won't get it on your actual handler).
  - `Dashboard` is for high-level overview of the API performance.

- If you click on a method, you will see method execution (request-response) cycle:
  ![](/md/231.jpg)

  - There is a `TEST` that you can test it in the dashboard (no need to Postman).
  - There are orange hints that you can disable or `show all hints`.
  - In `Method Request`,
    - we can define authorization.
    - Also, we can have `validators` related to `body` (by adding a model), `query parameters`, or `headers`.
      - Add model in the request body.
      - Choose the `Request Validator` as `Validate body`.
      - Note that you can have validation on the Lambda level.
    - Also, we can ask for `API Key`.
    - So it is like a gatekeeper.
  - In `Integration Request`,
    - We can modify the incoming request data (body, header, ...) and pass it on to the action method.
    - Scroll down to `Body Mapping Templates` and select `When there are no templates` for `Request body passthrough`.
    - Set the content type to `application/json` so requests with this type be handled by this template.
    - Create a template (the template is `Apache Velocity` template language):
    ```json
    {
        "age": $input.json('$.personData.age')
    }
    ```
    - You can also generate template from one of your models (for example the following will omit other properties sent to API Gateway):
    ```json
    #set($inputRoot = $input.path('$'))
    {
        "age": $inputRoot.age
    }
    ```
    - `$` is the request body.
    - `$input` is a variable provided by AWS to give you access to your request data such as body, params, ...
  - In `Integration Response`,
    - We can modify the response data to be sent to the client. We can define the response type, header (if defined in the `Method Response`) and map the body. So here we define the transformation.
    - We can add a template like:
    ```json
    {
        "your-age": $input.json('$') // gives us the data passed to the callback. here what Lambda returns is just a number
    }
    ```
    - You can also generate template from one of your models (for example the following will omit other properties returned from action method):
    ```json
    #set($inputRoot = $input.path('$'))
    {
        "your-age": $inputRoot
    }
    ```
  - In `Method Response`,
    - We define the shape of the possible responses (response types). For body, we use our models.
    - It is not binding/blocking any response to the client so if you only have a 200 status here, still any error will be thrown to the client.

- When you create a `resource`:

  - Do not check `configure as proxy resource` which means this resource should catch all other resources (this option if for MVC apps) -> you can put an MVC app behind a Lambda function and catch all the resources.
  - Enable API Gateway CORS -> It will create an `OPTIONS` method to handle OPTIONS requests which if you inspect its `Integration Response`, it sets `Access-Control-Allow-.` headers.

- Select the desired resource and then create a method for it and choose Lambda as `integration` type:

  - do not check `use Lambda proxy integration` which means it will send the whole incoming request with its metadata unfiltered as an event object to Lambda -> No separation of concerns. The logic should go to Lambda and authentication stuff should be on API Gateway side. If you check this option, you can't use `Integration Request` and `Integration Response`.
  - Select the region for your Lambda in the same region as your API (recommended).

- If you want to have a dynamic route,
  - create a resource on a given resource,
  - give it a name like id,
  - but in `Resource Path`, wrap it with {} like `{id}`.
  - Go to `Integration Request`
  ```json
  {
    "id": "$input.params('id')" // without " it will return a number, with this it will be a string
  }
  ```
  - Then the `event` on Lambda will have an `id` property.

# Lambda Functions

![](/md/232.jpg)

![](/md/233.jpg)

- To create a Lambda,

  - Select `Blank Function` as your `blueprint`.
  - Do NOT select a trigger. Instead, create a trigger-less Lambda. We will trigger the Lambda in API Gateway.
  - Name your function and choose a runtime.
  - You can write your code, paste it, upload a zip of your code, or upload it from S3.
  - You can pass environment variables.
  - If you have multiple functions in your code, the one that you export it is the entry point (you should also specify a different name if you change it from `index.handler`). If you upload it, the name should be `index.js` or you can rename it e.g. `app.handler`.
  - It receives three data: `event`, `context`, and `callback`:
    - `event`: the event data which depends on the event source. In case of API Gateway, we define the data passed to Lambda. If `Lambda proxy integration` is disabled, event is the body of the request but if that option is enabled, it will forward the whole request -> so we cannot return it as the response and we should return a response object like `{ headers: {'Access-Control-Allow-Origin':'*'}, body: {message:'hi'} }` as the second parameter of the callback.
    - `context`: the context that function is being executed, like the time that it has started, remaining time before the function times out, and ...
    - `callback`: itself receives two parameters. If the first one is anything but `null`, it means error and the second one will be omitted. The second one the data that we want to respond.
  - The `Role` is the permission that the Lambda needs (for example to access data). So choose `Create new role from template`, give it a name and leave policy template empty for now. Note that all functions have the permission to log by default.
  - You can add `Tags` to track your billing for example.
  - In `Advanced settings`, you can allocate the memory to Lambda and specify the `timeout duration` (choose a reasonable range because you don't want to pay for a lengthy error and on the other hand, you should not set it to low where it timesout before producing the result). 10s is good.
  - There is an option to push failed function calls on a queue to re-execute them.
  - You can have custom VPC (Virtual Private Cloud) to split your functions in different environments.
  - You can enable active tracing with `X-Ray` which is extra logging feature with extra money.
  - You can encrypt your environment variables.

- Now, go back to API Gateway and selects the Lambda function as the integration type of the method. We will prompted to give permission to API Gateway to invoke the function.
- Then deploy it to a new stage such as dev.
- It is true that we have enabled CORS for the resource but we have to adjust each method by going to `Method Response` and add `Access-Control-Allow-Origin` header and then go to `Integration Response` and set its value to `"*"`. Re-deploy. Another way is to select the method, and click on actions button and select `Enable CORS`. A much better way is to select the resource and `Enable CORS` for all methods in that resource.

- If you want to see logs for a Lambda (you have some `console.log`s), go to CloudWatch -> Logs -> Find the log group for the desired Lambda function -> Find the stream for each execution or executions grouped by being close to each other.

- Lambda can be invocated `synchronously` (API Gateway, Cognito) or `asynchronously` (S3). If a Lambda is invocated by the `SDK`, you can choose synchronously or asynchronously.
- Lambda events can be `push-based` (API, S3) or `pull/poll-based` (SQS, Dynamo, Kinesis).
- `context` has methods (getRemainingTimeInMillis) and properties (functionName -> can be useful to invoke the same Lambda function again programmatically in SDK, functionArn, awsRequestId, memoryLimitInMB, logGroupName, clientContext ...),

# DynamoDB

- It is a serverless key-value pair database (JSON-like).

![](/md/234.jpg)

![](/md/235.jpg)

![](/md/236.jpg)

- Go to `DynamoDB` and create a table:

  - give it a table name.
  - The primary key can be only the partition key (which can be string, number or binary).
  - It is convention to name it with PascalCase (like UserId).
  - Uncheck `use default settings` to see that you can define secondary indexes and also see the `Provisioned capacity`.

- After creation, you can go to `Items` tab and add an item from the AWS console by adding some attributes (PascalCase). You can also `scan` or `query` in that tab. `query` takes the condition into account whilst searching, but `scan` reads all data first.

- You can create tables not databases. You can have a different table in another region to mimic having a different database (dumb idea). Think Dynamo as separated S3 buckets which are not related.

- To use DynamoDB within a Lambda,
  - note that AWS SDK is provided to each Lambda by default. You have to `npm i aws-sdk` it in your local though.
  - note that you have to go to `IAM`, under `Roles` tab, select the role that you have given to your Lambdas and then see the `policies`. So you have to attach a policy (you can search for DynamoDB roles like full access and give it to your Lambda).
  - But as a best practice a Lambda that reads data should have read access and another Lambda a different DynamoDB access.
  - Note that policies are defined for the resource that we want to have access to. And then we will attach that policy to whatever we want.
  - Note that roles are defined for the things that want to have access to other things.
  - First you define the policy and then you define a role by attaching that policy.
  - So roles are for Lambda and policies are for DynamoDB here in this context.

```js
// it is better to put these imports and instantiating outside the function, because the wrapper for this function will be live for few minutes and it gives us performance benefits
const AWS = require("aws-sdk");
const dynamodb = new AWS.DynamoDB({
  region: "us-east-2",
  apiVersion: "2012-08-10",
});

exports.handler = (event, context, callback) => {
  const params = {
    Item: {
      UserId: {
        S: "user_" + Math.random(),
      },
      Age: {
        N: "" + event.age, // even here we have to pass a string and AWS will convert it to number because of N
        // a better solution is to make it a string in 'Integration Request'
      },
    },
    TableName: "table-name",
  };
  dynamodb.putItem(params, (error, data) => {
    if (error) {
      console.log(error);
      callback(error);
    } else {
      console.log(data);
      callback(null, data);
    }
  });
};
```

- To read one data use `getItem` and to get all data use `scan` methods on DynamoDB instance from SDk.
- To delete one data use `deleteItem`.
- If your Node version is 8 or higher, you can mark the function as async and just return the data and get rid of the callback.

# Authorization

- In `Method Request`, we have `Authorization`.

  - There is an option for `AWS_IAM ` which rarely used and means we want this API to be accessed via IAM credentials and therefore it is not public anymore.

- On the `Authorizers` tab for our selected API, we can create a `custom authorizer` or select a `cognito user pool authorizer`.

## Custom Authorizer

- If you have existing authorization system.

- By selecting `custom authorizer`, you should select a Lambda which will be called with some arguments from the incoming request (we select the source which is usually the Authorization header) and authenticates the user based on.

  - We need to create this Lambda, but note that this Lambda should return an `IAM Policy` like `{"Effect": "Allow", "Action": "execute-api"}` to allow or deny access, a `Principal ID` (User Id), and optionally a `Context` object `{"YourData":"YourValue"}`.

  ```js
  exports.handler = (event, context, callback) => {
    const token = event.authorizationToken;
    // Use token
    if (token === "allow") {
      const policy = genPolicy("allow", event.methodArn);
      const principalId = "aflaf78fd7afalnv";
      const context = {
        simpleAuth: true,
      };
      const response = {
        principalId: principalId,
        policyDocument: policy,
        context: context,
      };
      callback(null, response);
    } else if (token == "deny") {
      const policy = genPolicy("deny", event.methodArn);
      const principalId = "aflaf78fd7afalnv";
      const context = {
        simpleAuth: true,
      };
      const response = {
        principalId: principalId,
        policyDocument: policy,
        context: context,
      };
      callback(null, response);
    } else {
      callback("Unauthorized");
    }
  };

  function genPolicy(effect, resource) {
    const policy = {};
    policy.Version = "2012-10-17";
    policy.Statement = [];
    const stmt = {};
    stmt.Action = "execute-api:Invoke";
    stmt.Effect = effect;
    stmt.Resource = resource;
    policy.Statement.push(stmt);
    return policy;
  }
  ```

  - After creating Lambda and selecting it on Authorizer, setting the TTL (cache the result of Lambda so we don't hit Lambda too much) and creating it, you can assign it to `Method Request` of a method.
  - After assigning authorizer to our method, we can pass the `Principal Id` to the endpoint in `Integration Request`:

  ```json
  #set($inputRoot = $input.path('$'))
  {
      "age": $inputRoot.age,
      "userId": "$context.authorizer.principalId"
  }
  ```

## Cognito

![](/md/237.jpg)

- We have two options: `Manage your User Pools` and `Manage Federated Identities`. The latter is for when we want to use for example Google OAuth to grant access to a S3 bucket.
- `Manage your User Pools`:
  - Give it a name and choose `step through settings`.
  - In Cognito, by default users identified by `username` and `password` but you can check additional attributes. If you check `alias` it will replace the `username` by the checked attribute. Add `email` and make it alias for example.
  - Define password difficulty level.
  - Choose `Allow users to sign themselves up`.
  - You can add `MFA` as off, optional, or required.
  - You can make it mandatory to `require verification of emails or phones`: choose Email.
  - We can enable which device the user has logged in. The default is No.
  - If you want to connect a SPA, uncheck `Generate client secret`.

![](/md/238.jpg)

- To connect a front-end to the user pool,

  - you install the SDK on the front-end: `npm i amazon-cognito-identity-js`.

  ```js
  import {
    CognitoUserPool,
    CognitouserAttribute,
    CognitoUser,
    AuthenticationDetails,
  } from "amazon-cognito-identity-js";

  const POOL_DATA = {
    UserPoolId: "us-east-2-dfgdsfg", // can be found in General Setting in Cognito (just above the ARN for user pool)
    ClientId: "xfvcbdfgrgd4", // can be found under App Clients in Cognito console
  };

  const userPool = new CognitoUserPool(POOL_DATA);

  // in sign-up
  const attrList = [];
  attrList.push(
    new CognitouserAttribute({
      Name: "email",
      Value: user.email,
    })
  );
  userPool.signUp(
    user.username,
    user.password,
    attrList,
    null, // this for validating data on the server
    (err, result) => {
      // callback to be called when Cognito sends the response
      if (err) {
        // handle in the app
      } else {
        // handle in the app
        console.log(result.user);
      }
    }
  );

  // in user confirmation (we send a code to user's email)
  const cognitoUser = new CognitoUser({
    Username: username,
    Pool: userPool,
  });
  cognitoUser.confirmRegistration(code, true, (err, result) => {
    if (err) {
      // handle in the code
      return;

      // handle in the code
      // navigate away
    }
  });

  // in sign-in
  const authData = {
    Username: username,
    Password: password,
  };
  const authDetails = new AuthenticationDetails(authData);
  const cognitoUser = new CognitoUser({
    Username: username,
    Pool: userPool,
  });
  cognitoUser.authenticateUser(authDetails, {
    onSuccess(result) {
      // handle in the code
      //result = {
      //  accessToken: {
      //    jwtToken: "euJksdckjsbdcmsdc",
      //  },
      //  idToken: {
      //    jwtToken: "euJksdckjsbdcmsdc",
      //  },
      // refreshToken: {
      //    token: "euJksdckjsbdcmsdc",
      //  },
      //};
    },
    onFailure(err) {
      // handle in the code
    },
  });

  // in get current user
  const user = userPool.getCurrentUser(); // reaches to local storage
  user.getSession((err, session) => {
    // reaches to network: if the refresh token is valid, it generates new id and access token
    if (err) {
      //
    } else {
      if (session.isValid()) {
        //
      } else {
        //
      }
    }
  });

  // in logout
  userPool.getCurrentUser().signOut(); // remove the token stored in local storage bu Cognito

  // send a request
  const user = userPool.getCurrentUser().getSession((err, session) => {
    if (err) return;

    const jwt = session.getIdToken().getJwtToken();
  });
  ```

- To connect Cognito to the API gateway, under Authorizers -> Create -> Cognito User Pool Authorizer -> Choose your user pool, give it a name and create it. Then use it in `Method Request`.
- After assigning authorizer to our user pool, we can pass the `userId` to the endpoint in `Integration Request`:

  ```json
  #set($inputRoot = $input.path('$'))
  {
      "age": $inputRoot.age,
      "userId": "$context.authorizer.claims.sub"
  }
  ```

- You can also enforce an accessToken query parameter on you API Gateway (by making it required and validate it in `Method Request`) and then map it to the body to get access to it in your Lambda and then based on that access token, you can use Cognito SDK in your Lambda to get the user based on that access token.
- So by the help of access token, we can get the user in Lambda too (easier to get it from $context.authorizer.claims.sub in the body mapper).
- Id token will always be in the Authorization header.

# S3

![](/md/239.jpg)

- Files are objects and folders are bucket.
- We can enable versioning to keep track of object changes.
- Add all the build files to the bucket (do not change the default setting yet. No need to give public permission from here).
- Go to `Permissions` tab and create a ploicy to grant read-only access to anonymous user. It will open up our bucket to public but it won't serve it through HTTP.
- Go to `Properties` tab and enable static website hosting. Note that you should pass `index.html` for both entry and error since it is single page and should handle different routes.
- Just note that there is a server which we don't see.
- You can cerate another bucket for logs and then on the `Properties` tab of the main bucket, enable logging and choose the log bucket as target.

# CloudFront

![](/md/240.jpg)

- On CF, create a new distribution.
- Choose the S3 bucket as origin domain name.
- Choose `Customize` caching and enter a default TTL so that CF replaces the copies on that basis.
- You can compress the objects automatically.
- You can choose what edges you want to use: All edges.
- The default root object is `index.html`.
- You can turn on logging and select the same bucket for logs but different prefix.
- It will give you a URL at the end -> Use this instead of S3 URL.

# Route53

![](/md/241.jpg)

- Register a new domain on Route53 and pay for it.
- Go back to CF and set the purchased domain name as `Alternate Domain Names`: example.com and www.example.com.
- Go back to Route53 and go to `Hosted Zones`. Click on the purchased domain and then `Create Record Set`.
- Click on the `Alias`/`Yes` and select the CF target. Create another record set for www.example.com.

- Amazon `Aurora` is fully managed by Amazon Relational Database Service (`RDS`), which automates time-consuming administration tasks like hardware provisioning, database setup, patching, and backups.
- Amazon `Aurora Serverless` is an on-demand, auto-scaling configuration for Amazon `Aurora`. It automatically starts up, shuts down, and scales capacity up or down based on your application's needs. It enables you to run your database in the cloud without managing any database capacity.
