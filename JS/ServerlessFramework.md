- `AWS CDK` and `Serverless Framework` are comparable to each other as two `Infrastructure as Code` platforms.
- AWS Cloud Development Kit (AWS CDK) allows you to write your infrastructure code in Typescript. Your code eventually gets complied as a `CloudFormation` template and deployed as a CloudFormation Stack. we can create simple Serverless APIs with `AWS Lambda`, `DynamoDB` and `API Gateway` with AWS CDK and Typescript.
- Note that technically, you can deploy an `express` or a `Nest.js` app as a Lambda function but it is not a `native` serverless solution and there are many problems with it. So each route should be a different Lambda function.
- One drawback for me is that the local development environment is harder since you have to deploy each change. Even if you dockerize the serverless framework and use serverless-offline to create a HTTP server and invoke the handlers, there might be some problems with hot-module reloading (but try for yourself).

# Architecture

- Request comes through `API Gateway` and triggers `Lambda functions` after being authorized by another lambda function. They have some logic and save the data on serverless `DynamoDB` or `Amazon S3`.
- `AWS EventBridge` will run periodically and triggers another lambda function. That lambda function (to close the auction and send email to the highest bidder) sends a message to `AWS SQS` which will be picked up by another lambda function which uses `AWS SES` (Simple Email Service).

![](/md/174.jpg)

# Introduction

- It is not really serverless! It is a pricing model to focus on delivering functionality (pay-as-you-go).
- It is fully managed and you don't have to worry about `scaling`.
- We will use `Serverless Framework` to manage everything for us. It supports AWS, Azure, GCP, etc.
- The building blocks are `FaaS` (Function as a Service).
- Note that Lambda functions `need to be triggered`. Such as API Gateway, when uploading on S3, or inserting record in DynamoDB.
- There is a `serverless.yml` file which is the heart of serverless framework applications. You will define your cloud provider, resources, functions, events that triggers those functions and so on.
- Serverless forces you to follow `Infrastructure as Code` concept (just like Kuberenetes). There are lots of advantages such as having the infrastructure code side-by-side with the code (when you revert the code, you revert the infrastructure) and less room for human errors. For AWS, it actually uses `CloudFormation` for IaC.

# Installation

- Install AWS CLI version 2.
- Then `npm i -g serverless`.
- Create a user in AWS IAM with both programmatic and console access and attach a policy like `AdministratorAccess`.
- Then in the console, type `aws configure` and enter the access key id and secret access key.
- Type `serverless create --name PROJECT_NAME --template aws-nodejs` or `sls create --name PROJECT_NAME --template aws-nodejs-typescript` or any other template that you want and then `cd PROJECT_NAME` and then `npm install`.
- It will generate a `serverless.yml` file like this:

```yml
service:
  name: auction-service

plugins:
  - serverless-bundle
  - serverless-pseudo-parameters

provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256 # each function will have this memory. We can specify for each function too.
  stage: ${opt:stage, 'dev'} # stage is the option that will be provided to serverless CLI (--satge dev) with the default of dev
  region: eu-west-1 # the same as when configuring aws-cli (I want other ppl to deploy to this region as well not their default region in their aws cli)

functions:
  hello:
    handler: src/handlers/hello.handler # actually the file is .js in that location
    events:
      - http: # one of the events is http request
          method: GET
          path: /hello
```

The handler in `src/handlers/hello.js`:

```js
async function hello(event, context) {
  // you have access to the body of the request by using JSON.parse(event.body)
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello" }),
  };
}

export const handler = hello;
```

- To deploy the app for the very first time, type `sls deploy --stage dev -v` (-v to be verbose).
- After entering the above command, the plugins will kick in. For example, serverless-bundle will bundle our Javascript and then CF will start creating our stacks.
- If you want to remove your stack -> `sls remove` (It is the opposite of deploy).
- Whenever you make a change to `serverless.yml` file, you should re-deploy the whole project by `sls deploy` but if you change one function, you should just deploy that function with `sls deploy -f nameOfThatFunc`.
- By the way, if you haven't changed anything in the infra part, serverless framework is intelligent enough to not re-deploy it. So you can use, `sls deploy` anytime.
- An important note is that using `serverless offline` is not worth it. Because you have to mock a lot of services (like SQS) which are not official. It will not reflect how your application would run in Amazon. And note that most of the time, we deploy only functions which will take 3 seconds.

# DynamoDB

- To add a resource (such as DynamoDB), we add it in the `serverless.yml` file:

```yml
resources: # This is serverless syntax
  Resources: # This is CF syntax
    AuctionsTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: AuctionsTable
        BillingMode: PAY_PER_REQUEST # So we haven't provisioned read and write units
        AttributeDefinitions: # These are the attributes that must be present in each item (remember that DynamoDB is schema-less)
          - AttributeName: id
            AttributeType: S # string
        KeySchema:
          - AttributeName: id
            KeyType: HASH # So we need to specify the PK (partition key here)
```

- Then again we deploy the app using `sls deploy -v`.
- To interact with the DB, we need to install `aws-sdk`.

```js
import AWS from "aws-sdk";

const dynamodb = new AWS.DynamoDB.DocumentClient(); // it should be defined outside the scope of the function

async function createAuction(event, context) {
  //...

  try {
    await dynamodb
      .put({
        TableName: "AuctionsTable",
        Item: auction, // object with the id generated by uuid
      })
      .promise();
  } catch (error) {
    console.error(error);
  }

  // return
}
```

- Also we need to give the Lambda function the right to access to the DB (By now the policy is only giving access to CloudWatch for logging). So we go to `serverless.yml` file and add `iamRoleStatements` section:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: ${opt:stage, 'dev'}
  region: eu-west-1
  iamRoleStatements: # We could cut this section and paste it to each Lambda functions. Otherwise, it will be applied to all functions (like here).
    - Effect: Allow
      Action:
        # - dynamodb:* This is dangerous
        - dynamodb:PutItem
        - dynamodb:Scan # it is needed for getAuctions lambda function
        - dynamodb:GetItem # it is needed for getAuction lambda function
        - dynamodb:UpdateItem # it is needed for placeBid lambda function
        - dynamodb:Query # it is needed for processAuctions lambda function
      Resource: # We don't want to hard-code the arn of the table. Because we might want to deploy the project to another environment. So we will use that plugin.
        - arn:aws:dynamodb:#{AWS::Region}:#{AWS::AccountId}:table/AuctionsTable
        - !Join [
            "/",
            ["${self:custom.AuctionsTable.arn}", "index", "statusAndEndDate"],
          ] # We need to add the arn of the index table as well which its structure is like 'arn of the main table/index/indexName' so we are using this intrinsic function !Join. We didn't write it like that because for some reason it does'nt work when you try to invoke the function manually using sls invoke.
```

- the `#{}` will be interpreted by `serverless-pseudo-parameters` plugin.
- To define `getAuction` lambda function, note how we define the dynamic route param:

```yml
getAuction:
  handler: src/handlers/getAuction.handler
  events:
    - http:
        method: GET
        path: /auctions/{id}
```

- This `id` is accessible via `event.pathParameters` in the Lambda function.
- Also we will use `await dynamodb.get({TableName:process.env.AUCTIONS_TABLE_NAME, key: {id}}).promise()` which uses `query` and not `scan` behind the scene.

# Optimization of serverless.yml

- It is better to define the IAM policies and resources in a separate files. Also, the name of the tables should not be hard-coded in the Lambda functions. Because we are using different tables for different envs (dev, test, etc.).
- Create `resources/AuctionsTable.yml` and cut and paste the object in the `Resources` section.
- Create `iam/AuctionsTableIAM.yml` and cut and paste the object in the `iamRoleStatements` section. Add `AuctionsTableIAM:` in the first line and get rid of the `-`:

```yml
AuctionsTableIAM:
  Effect: Allow
  Action:
    - dynamodb:PutItem
  Resource:
    - arn:aws:dynamodb:#{AWS::Region}:#{AWS::AccountId}:table/AuctionsTable
```

- Then add `AuctionsTable: ${file(resources/AuctionsTable.yml):AuctionsTable}` in the `Resources` section. Note that the `:AuctionsTable` is the value of the object in the imported yml file.
- Then add `- ${file(iam/AuctionsTableIAM.yml):AuctionsTableIAM}`. So now you know why we added that `AuctionsTableIAM:` in the first line.
- Now, change the `TableName` in `resources/AuctionsTable.yml` to `TableName: AuctionsTable-${self:provider.stage}`. Note that ${self:...} is pointing to the `serverless.yml` file. So now the table name is dynamic based on the environment.
- Now, we have to change the table name in iam and resources. But it is better to define a single source of truth for that. So we define a custom configuration for it under `custom` section in the `serverless.yml` file:

```yml
custom:
  AuctionsTable:
    name: !Ref AuctionsTable # Ref is an intrinsic CF function to retrieve the DynamoDB table name (the 'AuctionsTable' is the logical name of the resource).
    arn: !GetAtt AuctionsTable.Arn
```

- Now, we can change `- arn:aws:dynamodb:#{AWS::Region}:#{AWS::AccountId}:table/AuctionsTable` to `- ${self:custom.AuctionsTable.arn}`.
- Now, to change the table name in the Lambda function, we define env variables in provider section (to let them be application level. We could have defined it per function):

```yml
provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: ${opt:stage, 'dev'}
  region: eu-west-1
  environment:
    AUCTIONS_TABLE_NAME: ${self:custom.AuctionsTable.name}
  iamRoleStatements:
    - ${file(iam/AuctionsTableIAM.yml):AuctionsTableIAM}
```

and then, use `process.env.AUCTIONS_TABLE_NAME` in the Lambda function.

# Middleware

- We will use `middy` as the middleware engine for AWS Lambda.
- `npm i ` the followings:

  - "@middy/core"
  - "@middy/http-cors"
  - "@middy/http-error-handler"
  - "@middy/http-event-normalizer"
  - "@middy/http-json-body-parser"
  - "@middy/validator"

- Then in the Lambda function:

```js
import middy from "@middy/core";
import httpJsonBodyParser from "@middy/http-json-body-parser";
import httpEventNormalizer from "@middy/http-event-normalizer";
import httpErrorHandler from "@middy/http-error-handler";
import cors from "@middy/http-cors";
import createError from "http-errors";

async function createAuction(event, context) {
  //...

  try {
    await dynamodb
      .put({
        TableName: process.env.AUCTIONS_TABLE_NAME,
        Item: auction,
      })
      .promise();
  } catch (error) {
    console.error(error);
    throw new createError.InternalServerError(error);
  }

  //...
}

export const handler = middy(createAuction)
  .use(httpJsonBodyParser()) // it automatically parses the body so we don't have to do it
  .use(httpEventNormalizer()) // will prevent us to access non-existing parameters such as query parameters
  .use(httpErrorHandler()) // it works with 'http-errors' package that we have installed
  .use(cors());
```

- Instead of importing all the middlewares in each Lambda function, we should define a `lib/commonMiddleware.js`:

```js
import middy from "@middy/core";
import httpJsonBodyParser from "@middy/http-json-body-parser";
import httpEventNormalizer from "@middy/http-event-normalizer";
import httpErrorHandler from "@middy/http-error-handler";
import cors from "@middy/http-cors";

export default (handler) =>
  middy(handler).use([
    httpJsonBodyParser(),
    httpEventNormalizer(),
    httpErrorHandler(),
    cors(),
  ]);
```

- Then, we can import it and use it like:

```js
export const handler = commonMiddleware(lambdaFunc);
```

- That `validator` middleware can be used to validate request based on some pre-defined schemas (JSON Schema Validation) like this:

```js
const schema = {
  properties: {
    body: {
      type: "object",
      properties: {
        title: {
          type: "string",
        },
      },
      required: ["title"],
    },
  },
  required: ["body"],
};

export default schema;
```

Then, import the schema and the middleware in the Lambda function and:

```js
export const handler = commonMiddleware(createAuction).use(
  validator({ inputSchema: createAuctionSchema })
);
```

# Repeating tasks by EventBridge

- The trigger for Lambda function is not the API Gateway anymore:

```yml
functions:
  processAuctions:
    handler: src/handlers/processAuctions.handler
    events:
      - schedule: rate(1 minute) # You can use cron as well
```

- If you want to see logs in real time, instead of going to CloudWatch, you can use `sls logs -f processAuctions -t`.
- Alternatively, you can comment out events above (to disable running it every minute in the dev env) and invoke the function by `sls invoke -f processAuctions -l` (`-l` to see the logs).
- The goal is to run processAuctions every minutes and if the current time is greater than the auction `endingAt` property, change the status of the auction item. To do that, the best approach is to create a `global secondary index key` based on the status as `partition key` and endingAt as `sort key` (scan would be very inefficient). So we add these attributes to the table:

```yml
AuctionsTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: AuctionsTable-${self:provider.stage}
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: id
        AttributeType: S
      - AttributeName: status
        AttributeType: S
      - AttributeName: endingAt
        AttributeType: S
    KeySchema:
      - AttributeName: id
        KeyType: HASH
    GlobalSecondaryIndexes: # here
      - IndexName: statusAndEndDate
        KeySchema:
          - AttributeName: status
            KeyType: HASH
          - AttributeName: endingAt
            KeyType: RANGE
        Projection:
          ProjectionType: ALL # It saves all other attributes in the index table as well
```

# Authorization with Auth0

- Sign-up with `Auth0`. Sign-in to the console and create an application and chose `SPA` as type of the app.
- In `Allowed Callback URLs`, `Allowed Web Origins`, and `Allowed Logout URLs`, enter `http://localhost:3000` which is for the react frontend.
- Then in `Advanced Settings`, on `Grant Types` tab, mark `Password` and click on `Save`.
- Then in `Tenant Settings`, make sure that the `Default Directory` is `Username-Password-Authentication` and then `Save`.
- Then you can create a user in the Auth0 dashboard and choose Username-Password-Authentication for the `connection`. We will use this user and password to send it to Auth0 and get jwt back.
- Deploy the Auth service (read the doc in my github repo).
- To protect the routes in the Auction service, add the ARN of the authorizer Lambda function in the custom section:

```yml
custom:
  authorizer: arn:aws:lambda:#{AWS::Region}:#{AWS::AccountId}:function:auth-service-${self:provider.stage}-auth
```

and then for each Lambda function, add authorizer:

```yml
createAuction:
  handler: src/handlers/createAuction.handler
  events:
    - http:
        method: POST
        path: /auction
        cors: true
        authorizer: ${self:custom.authorizer}
```

- Then the claims will be on `event.requestContext.authorizer` in each function.

# Notification with Simple Email Service (SES)

- Create the template by `sls create --name notification-service --template-url https://github.com/codingly-io/sls-base` and then cd into it and `npm i`.
- Add the region in `serverless.yml` file.
- Go to `SES` on the AWS dashboard and on `Email Addresses` tab and verify the email address that you want to send email from (AWS will send an email to verify).
- Create a Lambda function in the notification service to send emails (we will trigger it manually now but soon we will rely on SQS).

```js
async function sendMail(event, context) {
  console.log(event);
  return event;
}

export const handler = sendMail;
```

- Give it the permission to access SES by creating `iam/SendMailIAM.yml`:

```yml
SendMailIAM:
  Effect: Allow
  Action:
    - ses:SendEmail
  Resource: arn:aws:ses:*
```

and in `serverless.yml`:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 128
  region: us-east-1
  stage: ${opt:stage, 'dev'}
  iamRoleStatements:
    - ${file(iam/SendMailIAM.yml):SendMailIAM}

functions:
  sendMail:
    handler: src/handlers/sendMail.handler
```

- Then deploy `sls deploy -v` and invoke by `sls invoke -f sendMail -l`.
- In this template, the `aws-sdk` has been installed. So change the Lambda function to:

```js
import AWS from "aws-sdk";

const ses = new AWS.SES({ region: "us-east-1" });

async function sendMail(event, context) {
  const params = {
    Source: "john.doe@gmail.com",
    Destination: {
      ToAddresses: ["john.doe@gmail.com"],
    },
    Message: {
      Body: {
        Text: {
          Data: "Hi there",
        },
      },
      Subject: {
        Data: "Test Mail",
      },
    },
  };

  try {
    const mailResult = await ses.sendEmail(params).promise();
    console.log(mailResult);
    return result;
  } catch (error) {
    console.error(error);
  }
}

export const handler = sendMail;
```

- Now, we need to add the SQS resource in notification service: `resources/MailQueue.yml`:

```yml
MailQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: ${self:custom.mailQueue.name}
```

and in `serverless.yml`:

```yml
resources:
  Resources:
    MailQueue: ${file(resources/MailQueue.yml):MailQueue}

functions:
  sendMail:
    handler: src/handlers/sendMail.handler
    events:
      - sqs:
          arn: ${self:custom.mailQueue.arn}
          batchSize: 1 # Process one message at a time. If you have a lot of traffic, increase it. The default batch size (max) is 10.

custom:
  mailQueue:
    name: MailQueue-${self:provider.stage}
    arn: !GetAtt MailQueue.Arn # We can use the logical name for CF in 5 line above to get the ARN
```

and in `iam/MailQueueIAM.yml`:

```yml
MailQueueIAM:
  Effect: Allow
  Action:
    - sqs:ReceiveMessage
  Resource: ${self:custom.mailQueue.arn}
```

- Then, change the Lambda function to:

```js
import AWS from "aws-sdk";

const ses = new AWS.SES({ region: "us-east-1" });

async function sendMail(event, context) {
  const records = event.Records; // It has one message because we specified batch size of 1

  let result = [];
  for (let i = 0; i < records.length; i++) {
    console.log("record processing", records[i]);

    const email = JSON.parse(records[i].body);
    const { subject, body, recipient } = email;

    const params = {
      Source: "guilherme.eti@gmail.com",
      Destination: {
        ToAddresses: [recipient],
      },
      Message: {
        Body: {
          Text: {
            Data: body,
          },
        },
        Subject: {
          Data: subject,
        },
      },
    };

    try {
      const mailResult = await ses.sendEmail(params).promise();
      result.push(mailResult);
    } catch (error) {
      console.error(error);
    }
  }
  console.log("sendMail finished", result);
  return result;
}

export const handler = sendMail;
```

- Now, we need to know the URL and ARN of the message queue to be used in the auction service. We could hard-code it or use pseudo parameters but we will use `outputs`. With output, we can export variables from one CF stack and import it in another stack. Because we need to add a policy to auction service Lambda functions to send message to this queue. This practice is also good in the sense that our stacks are dependent and if we deploy auction service and forget to deploy the notification service, we get error. So we change the `resources/MailQueue.yml`:

```yml
MailQueue:
  Type: AWS::SQS::Queue
  Properties:
    QueueName: ${self:custom.mailQueue.name}

Outputs:
  MailQueueArn:
    Value: ${self:custom.mailQueue.arn}
    Export:
      Name: ${self:custom.mailQueue.name}-Arn
  MailQueueUrl:
    Value: ${self:custom.mailQueue.url}
    Export:
      Name: ${self:custom.mailQueue.name}-Url
```

and `serverless.yml`:

```yml
resources:
  Resources:
    MailQueue: ${file(resources/MailQueue.yml):MailQueue}
  Outputs:
    MailQueueArn: ${file(resources/MailQueue.yml):Outputs.MailQueueArn}
    MailQueueUrl: ${file(resources/MailQueue.yml):Outputs.MailQueueUrl}

functions:
  sendMail:
    handler: src/handlers/sendMail.handler
    events:
      - sqs:
          arn: ${self:custom.mailQueue.arn}
          batchSize: 1

custom:
  mailQueue:
    name: MailQueue-${self:provider.stage}
    arn: !GetAtt MailQueue.Arn
    url: !Ref MailQueue
```

- We then deploy and we have two stack outputs. To import it in the auction service, we define it in custom section and pass it as env variable to our Lambda functions. We also need to define a policy to permit functions to send messages to the queue:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: ${opt:stage, 'dev'}
  region: us-east-1
  environment:
    AUCTIONS_TABLE_NAME: ${self:custom.AuctionsTable.name}
    MAIL_QUEUE_URL: ${self:custom.MailQueue.url}
  iamRoleStatements:
    - ${file(iam/AuctionsTableIAM.yml):AuctionsTableIAM}
    - ${file(iam/MailQueueIAM.yml):MailQueueIAM}

#...

custom:
  authorizer: arn:aws:lambda:#{AWS::Region}:#{AWS::AccountId}:function:auth-service-${self:provider.stage}-auth
  AuctionsTable:
    name: !Ref AuctionsTable
    arn: !GetAtt AuctionsTable.Arn
  MailQueue:
    arn: ${cf:notification-service-${self:provider.stage}.MailQueueArn} # variable inside variable
    url: ${cf:notification-service-${self:provider.stage}.MailQueueUrl}
```

and the `iam/MailQueueIAM.yml`:

```yml
MailQueueIAM:
  Effect: Allow
  Action:
    - sqs:SendMessage
  Resource: ${self:custom.MailQueue.arn}
```

# Uploading pictures to S3

- In custom section of `serverless.yml`, define the name of the bucket for different envs:

```yml
custom:
  authorizer: arn:aws:lambda:#{AWS::Region}:#{AWS::AccountId}:function:auth-service-${self:provider.stage}-auth
  AuctionsTable:
    name: !Ref AuctionsTable
    arn: !GetAtt AuctionsTable.Arn
  MailQueue:
    arn: ${cf:notification-service-${self:provider.stage}.MailQueueArn}
    url: ${cf:notification-service-${self:provider.stage}.MailQueueUrl}
  AuctionsBucket:
    name: course-serverless-framework-sdfsdfghdfgn-${self:provider.stage} # it is global (not region) and needs to be unique
```

Note that, because we want everyone to be able to read from our bucket, we will create a policy as well. In `resources/AuctionsBucket.yml`:

```yml
AuctionsBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: ${self:custom.AuctionsBucket.name}
    LifecycleConfiguration: # objects will be deleted after 1 day (you can remove this section)
      Rules:
        - Id: ExpirePictures
          Status: Enabled
          ExpirationInDays: 1

AuctionsBucketPolicy:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket: !Ref AuctionsBucket
    PolicyDocument:
      Statement:
        - Sid: PublicRead
          Effect: Allow
          Principal: "*"
          Action:
            - s3:GetObject
          Resource: arn:aws:s3:::${self:custom.AuctionsBucket.name}/*
```

Then again in `serverless.yml`:

```yml
resources:
  Resources:
    AuctionsTable: ${file(resources/AuctionsTable.yml):AuctionsTable}
    AuctionsBucket: ${file(resources/AuctionsBucket.yml):AuctionsBucket}
    AuctionsBucketPolicy: ${file(resources/AuctionsBucket.yml):AuctionsBucketPolicy}
```

and also we want to authorize our Lambda function to write to the bucket so `iam/AuctionsBucketIAM.yml`:

```yml
AuctionsBucketIAM:
  Effect: Allow
  Action:
    - s3:PutObject
  Resource: arn:aws:s3:::${self:custom.AuctionsBucket.name}/*
```

and for the third time in `serverless.yml`:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  memorySize: 256
  stage: ${opt:stage, 'dev'}
  region: us-east-1
  environment:
    AUCTIONS_TABLE_NAME: ${self:custom.AuctionsTable.name}
    MAIL_QUEUE_URL: ${self:custom.MailQueue.url}
    AUCTIONS_BUCKET_NAME: ${self:custom.AuctionsBucket.name}
  iamRoleStatements:
    - ${file(iam/AuctionsTableIAM.yml):AuctionsTableIAM}
    - ${file(iam/MailQueueIAM.yml):MailQueueIAM}
    - ${file(iam/AuctionsBucketIAM.yml):AuctionsBucketIAM}
```

Now, we need a Lambda to upload picture:

```yml
functions:
  uploadAuctionPicture:
    handler: src/handlers/uploadAuctionPicture.handler
    events:
      - http:
          method: PATCH
          path: auction/{id}/picture
          cors: true
          authorizer: ${self:custom.authorizer}
```

- Now we can work on the function which we will send the picture as base64 body and
  - we search for the id to be existing
  - we check that the auction seller is the requester
  - we will update the picture to s3 by creating buffer out of base64 and get back the URL
  - we will update the table and set the picture URL
