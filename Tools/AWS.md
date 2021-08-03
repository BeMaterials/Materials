# Basic

## AWS Regions

- AWS has regions all around the world: `us-east-1`.
- Each region has availability zones (AZs): `us-east-1a`.
- You can go up to six AZs in a region (a, b, c, d, e, f).
- Each availability zone is a physical data center in a region and they are separated from each other.
- AWS Consoles are `region-scoped` except `IAM` and `S3`.

## IAM (Identity and Access Management)

- Your whole AWS security is here: Users, Groups, Roles.
- `Root` account should `never` be used.
- IAM is at the center of AWS.
- `Users` are physical persons and must be created with proper permissions.
- `Groups` contain users and should be based on functions (admins, devOps, ...).
- `Roles` are going to be given to a machine.
- We have `policies` which are written in `JSON` and define what each `Users`, `Groups` and `Roles` can and cannot do (permissions).
- IAM has predefined `managed policies`.
- `Least privilege principle`: it's best to give users the minimal amount of permissions they need to perform their job.
- `One` IAM `User` per physical `person` and `one` IAM `Role` per `application`.
- Create a user that has admin access (belong to a group like Admin with admin policy attached) and use that instead of root account.

## EC2

- It mainly consists in the capability of:
  - Renting virtual machines (EC2).
  - Storing data on virtual drives (EBS).
  - Distributing load across machines (ELB, Elastic Load Balancing).
  - Scaling the services using an auto-scaling group (ASG).
- It is necessary to know `serverless` later.

### SSH

- SSH allows you to control a remote machine, all using the command line.
- We had an SSH security group on our EC2 instance, and we allowed SSH on port 22 from any IP.

```sh
ssh -i C:\path\to\the\pem\file.pem ec2-user@the-public-ipv4-of-ec2-instance
ssh -i C:\Users\Ben\Downloads\EC2tutorial.pem ec2-user@ec2-3-106-165-72.ap-southeast-2.compute.amazonaws.com

[ec2-user@ip-172-31-12-4 ~]$
```

- The `@ip-172-31-12-4` is the private IP.
- If you have VPN, you can use the private IP (?).
- Also, you can connect by using EC2 Instance Connect (browser-based SSH connection) in EC2 console. This only works for Amazon Linux 2 AMIs (Amazon Machine Image).

### Security Groups

- It is basically a set of firewall rules on EC2 instances.
- It is attached to our EC2 instance and control the inbound and outbound traffic (access to ports and authorized IP ranges).
- They can be attached to multiple instances and an instance can have multiple security groups.
- They are tied to `region/VPC` combinations.
- They live outside the EC2- if traffic is blocked, the EC2 instance won't see it.
- It is good to maintain one separate SG for SSH access.
- Most needed skill to learn to troubleshoot networking issues. If your application is not accessible (`timeout`), it's a security group issue. But, if your application gives a `connection refused` error, then it is an application error or it's not launched.
- All inbound traffic is blocked by default.
- All outbound traffic is authorized by default.
- We can authorize inbound traffic from other EC2 instances by referring their SGs in the inbound rules.

### IP

- When you stop and then start an EC2 instance, it can `change` its `public IP` (the private IP is for internal AWS network and remains the same).
- If you need to have a fixed public IP, you need an `elastic IP`.
- An elastic IP is a public IPv4 IP which you own as long as you don't delete it.
- You can attach it to one EC2 instance at a time, obviously.
- It is uncommon to use elastic IPs to mask failure of an instance; Instead, use a random public IP and register a DNS name to it.
- With load balancers, we don't need to use public IPs at all which is the best pattern.

### Installing Apache on EC2

- After SSHing to the EC2 instance, run the following to elevate to the root account:

```sh
sudo su
```

- Then run the following to update all packages in the machine:

```sh
yum update -y
```

- Then run the following to install `httpd`:

```sh
yum install -y httpd.x86_64
```

- Run the following to start the service and to ensure that the system remains enabled across reboots:

```sh
systemctl start httpd.service
systemctl enable httpd.service
```

- curl is basically to load whatever is in this URL (you will see a giant html):

```sh
curl localhost:80
```

- To access that from outside, we have to open port 80 of EC2 to the world in the inbound traffic of the SG attached to this EC2.
- So we have to add some content to `var/www/html` folder:

```sh
echo "Hi there" > /var/wwww/html/index.html
```

- To see the private DNS (which includes the private IP):

```sh
hostname -f
```

### EC2 User Data

- It is possible to bootstrap (launching commands when a machine starts) our instance using an `EC2 User data` script:
  - Install updates
  - Install software
  - Download common files from the internet
  - Anything you can think of
- Example:
  - Terminate the current instance.
  - Launch a new one.
  - In the `Configure Instance Details` step, scroll down to the `Advanced Details` section and you can see `User data`.
  - Write the commands in the box and remember that User Data is automatically run with the sudo command (so no need for sudo su).
  ```sh
  #!/bin/bash
  yum update -y
  yum install -y httpd.x86_64
  systemctl start httpd.service
  systemctl enable httpd.service
  echo "Hi there from $(hostname -f)" > /var/wwww/html/index.html
  ```
  - In `Configure Security Group` step, choose the previous SG (even we deleted the EC2, the SG remained).
  - At the end choose an existing key pair and acknowledge that you have access to the .pem file and launch the instance.
- You can build custom AMI (OS with some modifications like pre-installed packages) instead of using EC2 User Data. Custom AMIs are faster at boot time. Note that they are built for a specific region.

### EC2 Types

- `On Demand`:
  - pay for what you use (billing per second)
  - has the highest cost but no upfront payment
  - no long term commitment
- `Reserved`:
  - 1 or 3 years
  - up to 75% discount compared to on-demand
  - pay upfront for what you use with long term commitment
  - recommended for steady state usage application (if you have database)
  - There are some variations like `convertible` which you can change the type but has less discount and `scheduled` which will be launched within time window you reserve
- `Spot instances`:
  - up to 90% discount
  - you bid a price and get the instance as long as it's under the price
  - useful for batch jobs, Big Data analysis, or ...
- `Dedicated Hosts`:
  - physical dedicated EC2 server for your use
  - more expensive

## ELB (Elastic Load Balancing)

- Load balancer is a `server` that forwards internet traffic to multiple servers (`EC2` instances) downstream.
- The benefit is a single point of access (DNS) to the app but the load will be spread across multiple instances.
- So the URL for the ELB will never change.
- ELB does regular health checks (sends a request to an endpoint on a port that we have specified like /health:4567 -> not 200 -> unhealthy) to the instances and if one of them is not, it redirects the traffic to healthy ones.
- ELB provides SSL termination (HTTPS) for your website (the termination and the encryption of the connection is between the client and the ELB and the ELB and EC2 instances talk in HTTP with each other).
- Because ELB can be across zones in a region it provides high availability.
- There are classic load balancer (v1), application load balancer (v2), and network load balancer (v2).
- You can setup internal (private) or external (public) ELBs.
- ALB or Application load balancer (v2) or layer 7
  - They can load-balance to multiple HTTP applications across machines (target group)
  - They can load-balance to multiple applications on the same machine (containers)
  - They can load-balance based on route (hostname/path) (send all Route/user to one `target group` and send all Route/search to another target group) in URL
  - They are awesome for microservices & container-based applications (Docker & Amazon ECS)
  - They have a port mapping feature to redirect to the same instance of the application running on the same machine (?).
  - The application servers don't see the IP of the client directly (the true IP of the client is inserted in the header X-Forwarded-For)
  - They are for HTTP/HTTPS/Websocket
- NLB or Network load balancer (v2) or layer 4
  - They are similar to ALBs but instead of redirecting different HTTP requests, they are redirecting different TCP traffics to different target groups.
- CLB and ALB support SSL certificates and provide SSL termination.
- Any load balancer has a static host name (we get a URL). Do not resolve and use underlying IP.
- NLB directly sees the client IP.
- The setup is to create a load balancer, create an SG for it, create a target group for it and register the instances to the target group. Do not forget to change the inbound traffic of the EC2 instances to be only from the SG of the load balancer.

## ASG (Auto Scaling Group)

- It can scale out (add EC2 instances) to match an increased load.
- It can scale in (remove EC2 instances) to match a decreased load.
- We can ensure we have a minimum and maximum number of machines running.
- We can automatically register new instances to a load balancer.
- ASG and ELB really work hand in hand together.
- ASGs have the following attributes:
  - `Launch configuration` (just like what we did manually)
    - AMI + instance type
    - EC2 user data
    - EBS volumes
    - SGs
    - SSH key pair
  - Min size/max size/initial capacity/desired capacity
  - Network (VPC) + subnet (AZ) information (in which our ASG will be able to create instances. Choose all AZs in a VPC to be highly available)
  - Load balancer information (target group information) -> In `Advanced Details` sections, mark `Receive traffic from one or more load balancers` and select the target group, which means that every instance that will be created, will be registered to that target group.
  - Scaling policies (what will trigger a scale out or in)
  - IAM roles attached to an ASG will get assigned to EC2 instances.
- ASGs are free and you only pay for underlying resources being launched.
- ASG can terminate instances marked as unhealthy by a LB and hence replace them.
- Having instances under an ASG means that if they get terminated for whatever reason, the ASG will restart them so extra safety.
- It is possible to scale an ASG based on `CloudWatch alarms` (based on the alarm, we can create scale-out or in policies).
  - An alarm monitors a metric (such as average CPU).
  - Metrics are computed for the overall ASG instances.
- It is also possible to define better`auto scaling rules that are directly managed by EC2`:
  - Target average cpu usage
  - Number of requests on the ELB per instance
  - Average network in
  - Average network out
- Also, we can auto scale based on a `custom metric` such as connected users which we should program in the application on EC2 to send metric to CloudWatch (PutMetric API), create alarm to react to low/high values, and then use that alarm as the scaling policy.

## EBS Volume (Elastic Block Store)

- When we terminate EC2, it loses its root EBS volume (we can disable it).
- An EBS volume is a network drive (not physical drive) you can attach your instance while they run. They can be attached only to one instance at a time.
- It is locked to an AZ. An EBS volume in one AZ cannot be attached to an instance in another AZ unless we first snapshot it.
- They have provisioned capacity (size in GB and IOPs {Input Output per second}).
- When you create an EC2 instance a GP2 (general purpose SSD) volume is assigned by default.
- EBS volumes can be backed up using snapshots which take the used space (not all) of the blocks on the volume.
- Because EBS backups use IO, you should not run them while your application is handling a lot of traffic.
- Snapshots are also used when volume migration: resizing a volume down, changing the volume type, encrypt a volume.
- We can create an encrypted EBS volume: data at rest is encrypted, data in flight (between instance and the volume) is encrypted, all snapshots are encrypted, all volumes created from an encrypted snapshot is encrypted. Encryption has a minimal impact on latency so you should use it.
- We have also `instance store` in comparison with EBS which are physically attached to the machine and good for very high performance but the instance store will be lost upon termination, you can't resize them, and backups must be operated by the user.

## Route 53

- Is a managed DNS.
- In AWS, the most common records are:
  - A: URL to IPv4
  - AAAA: URL to IPv6
  - CNAME: URL to URL
  - Alias: URL to AWS resource.
- Every web browser knows how to work with a DNS, to send a request to route 53 (DNS server), get back the IP, send the HTTP request to the IP, and cache the mapping.
- Route 53 can use public domain names you own or private domain names that can be resolved by your instances in your VPC.
- Use Alias records over CNAME for AWS resources.
- Register a domain via Route 53, a hosted zone will be created automatically for you, check the DNS name of the load balancer, back to Route 53 console `Create Record Set` for the created hosted zone, choose `A - IPv4 address` as the Type and `Yes` Alias, choose the DNS name of the load balancer as the alias target, and click on create.

## RDS (Relational Database Service)

- It allows you to create database in the cloud that is managed by AWS: Postgres, Oracle, MySQL, MariaDb, MS SQL Server, Aurora (AWS proprietary database).
- Advantages over using DB on EC2 with EBS
  - OS patching
  - continuous backups and restore to specific timestamp
    - daily snapshot of the database (7 days retentions which can be increased to 35 days)
    - capture transaction logs in real time (so the ability to restore to any point in time)
    - you can also manually trigger a snapshot and keep the backup for as long as you want.
  - monitoring dashboards
  - read replicas for improved read performance:
    - up to 5 read replicas
    - within AZ, cross AZ, or cross region
    - it is asynchronous (eventual consistency, small delay) between the master and replicas
    - replicas can be promoted to their own DB
    - application must update the connection string to leverage read replicas
  - multi AZ setup for disaster recovery (DR)
    - app only talks to the master (in contrast to the read replicas). So only one DNS name is exposed to your app.
    - it is synchronous replication to another instance in another AZ which is called standby.
    - automatic fail-over from the master to the standby in there are any issues. Which is completely automatic.
    - not used for scaling, it is for DR.
  - maintenance windows for upgrades
  - scaling capability
  - Encryption
    - You can have encryption at rest. Also, you can enforce SSL to encrypt data to RDS in flight. So no user can connect to the db unless on SSL connection.
  - Security
    - RDS databases are usually deployed within a private subnet.
    - We use SGs to communicate with RDS.
    - IAM policies help control who can manage AWS RDS.
  - BUT you can't SSH into your RDS instances

## ElastiCache

- is to get managed Redis or Memcached.
- caches are in-memory databases with really high performance and low latency.
- helps reduce load off of DB for read intensive workloads.
- It has write scaling (using sharding), read scaling (using read replicas), and multi AZ capabilities.
- AWS takes care of OS maintenance, patching, optimization, setup, configuration, monitoring, failure recovery and backups.
- Applications should be written in a way that it queries ElastiCache, if not available, get from RDS and store in ElastiCache (lazy loading approach). It should also have an invalidation strategy to make sure only the most current data is used. It can be in the application or by using TTL (e.g. 5 mins).
- Write through approach is when any update to the database should update the cache as well. In lazy loading we have read penalty (in case of cache miss we have to make three calls) but in write through, there is a write penalty (we have to make two calls to write). In this approach we will have missed data until it is added or updated, so We can combine these two approaches. Another disadvantage is a lot of data that will never be read.
- Another pattern is to write session data into ElastiCache and if user hits another instance of application in a ASG, the instance retrieves the data and the user is already logged-in.
- Redis is an in-memory key value store and it has persistence too (unlike Memcached). It is great to host:
  - user sessions
  - leaderboard (for gaming)
  - relieve pressure from RDS
  - Pub/Sub capability for messaging

## VPC (Virtual Private Cloud)

- Within a region, you can create a VPC.
- A default VPC (and one default subnet per AZ in the VPC region) has been created when you sign up.
- Each VPC contains subnets (networks).
- So VPCs are per account per region.
- In each AZ, it is common to create two subnets: public (load balancers, static websites, files, public authentication layer) and private (web applications, databases).
- Public and private subnets can communicate if they are in the same VPC.
- You can also have many subnets in each AZ.
- It is possible to use a VPN to connect to a VPC and and access all the private IPs straight from your laptop.
  ![](/md/3tiers.jpg)

## S3

- `Infinitely scaling` storage.
- Many AWS services uses S3 as an integration as well.

### Buckets

- S3 allows people to store objects (files) in the buckets (directories).
- Buckets are `regional` services but they must have `globally unique name`.
- Naming conventions:
  - no uppercase
  - no underscore
  - 3-63 characters long
  - not an IP
  - must start with lowercase letter or number

### Objects

- Objects (files) have a key. The `key` is the `full path` in the bucket (so there is no concept of directories within buckets -> just keys with very long names that contain slashes).
- Object values are the content of the file:
  - max size is 5TB
  - if uploading more than 5 GB, must use `multi-part upload`
- Objects can have metadata which is a list of key value pairs
- Objects can have tags which is a list of key value pairs (tp to 10)
- Objects can have version Id, if versioning is enabled
  - versioning is enabled at the bucket level.
  - same key overwrite will increment the version: kjhkmnb, lkjhqwd, ...
  - it is best practice to version your buckets so you can roll back to previous version
  - any file that is not versioned prior to enabling versioning will have version `null`
  - if you delete a file when versioning is enabled, a delete marker will just be assigned to it
  - you can hide or show different versions of objects
- Objects can be encrypted (one by one or at the bucket level):
  - there are 4 methods of encrypting objects:
    - SSE-S3:
      - encryption using keys handled & managed by AWS S3.
      - must set header: "x-amz-server-side-encryption":"AES256".
    - SSE-KMS:
      - encryption using keys handled & managed by KMS. So you have more control over the rotation of keys.
      - object is encrypted server side like above.
      - must set header: "x-amz-server-side-encryption":"aws:kms".
    - SSE-C:
      - encryption using keys handled & managed by the customer
      - HTTPS must be used in this case because encryption key must be provided for every request made.
      - Amazon S3 does not store the encryption key you provide and after encryption in the server side, amazon will destroy the key.
    - Client side encryption:
      - Client must encrypt data before sending to S3 using S3 encryption SDK.
      - Client must decrypt data when retrieving from S3.
    - Encryption in transit (SSL):
      - AWS S3 exposes HTTP endpoints for non encrypted and HTTPS endpoints for encryption in flight which is recommended.

### Permissions

- There are two ways to enforce permissions:
  - User based:
    - Using IAM policies to restrict what a user can do with his S3 console.
  - Resource based (much more popular):
    - Using bucket policies which is a JSON based policy:
      - Resources: buckets and objects the policy is applied to (copy the Amazon Resource Name, ARN of the bucket/\*)
      - Actions: set of API to allow or deny (e.g. PutObject)
      - Effect: allow/deny (e.g. Deny)
      - Principal: the account or user to apply the policy to (e.g. \*)
      - Condition: which has a condition, a key and a value (e.g. Condition: StringNotEquals, Key: s3:x-amz-server-side-encryption, and Value: AES256)
    - Use S3 bucket policy to:
      - grant public access to a bucket
      - force objects to be encrypted at upload
      - grant access to another account (cross account)
- Logging and audit:
  - S3 access logs can be stored in other S3 bucket (otherwise recursion happens!).
  - API calls can be logged in AWS CloudTrail.
- User security:
  - MFA can be required in versioned buckets to delete objects
  - signed URLs which are valid only for a limited time (e.g. premium video service for logged in users)

### S3 Websites

- S3 can host static websites and have them accessible on the WWW.
- The website URL will be: `<bucket-name>.s3-website.<AWS-region>.amazonaws.com`.
- If you get a 403 (forbidden) error, make sure that the bucket policy allows public reads (GetObject).

### S3 CORS

- If you request data from another S3 bucket (e.g. imagebucket), you need to enable CORS (in the imagebucket).
- CORS allows you limit the number of websites that can request your files in S3 (and limit your costs).
- Browser sends a request with a header `ORIGIN:http://originbucket.s3-website.eu-west-3.amazonaws.com`, and the `imagebucket` sees the origin and if it is enabled it sends with a header `Access-Control-Allow-Origin: <domain>`.

### S3 Consistency

- Read after write consistency for PUTs of new objects (it is PUT not POST) -> as soon as an object is written, we can retrieve it.
- Eventual consistency for DELETEs and PUTs of existing objects -> if we read an object after updating/deleting, we might get the older version.

### S3 Performance

- Historically, it is recommended to put 4 random characters in front of your key name (folders' name basically) to optimize performance.
- Faster upload of large objects (>=100MB), use multipart upload -> parallelizes PUTs for greater throughput, maximizes your network bandwidth, decreases time to retry in case a part fails, it is a must when object size is greater than 5GB.
- Use `CloudFront` to cache S3 objects around the world (improves reads).
- Use `S3 Transfer Acceleration` (uses the same edge locations as the CloudFront does). You upload to a near region and it sends it to another region (improve writes). So you change the endpoint to upload and the performance improves.
- If SSE-KMS encryption is enabled, you might be limited to your AWS limits for KMS usage (it can't keep up with S3 service).

### S3 Select

- It might be the case that you want just a subset of data from S3.
- With S3 Select, you can use SQL SELECT queries to let S3 know exactly which filters you want.
- It works with files in CSV, JSON, or Parquet format (even if they are compressed with GZIP or BZIP2).
- No sub-queries or joins are supported.
- `select * from s3object s where s.\"Country\" like '%United States%'`.

## CloudFront

- Content Delivery Network (CDN).
- Improves read performance, content is cached at the edge.
- There are 136 points of presence globally (edge locations).
- Popular with S3 but works with EC2 and load balancing as well.
- Can help protect against network attacks such as DDOS.
- Can provide SSL encryption (HTTPS) at the edge using ACM (Amazon Certificate Manager).
- CloudFront can use SSL encryption (HTTPS) to talk to your application.
- Supports RTMP protocol for video/media.

# Developing on AWS

## CLI

- How to perform interactions with AWS without using the AWS console?
- How to interact with AWS proprietary services? (S3, DynamoDB, etc.)
- Performing AWS tasks can be done in several ways:

  - Using AWS CLI on our local computer

    - First download and install AWS CLI for your OS.
    - Select you user from IAM and go to `Security credentials` to create Access keys (download the CSV).
    - in the console of our local machine:

    ```sh
    aws configure
    <Access key>
    <Secret access key>
    <Region>
    <none>
    ```

    - The above creates a folder `.aws` with two files `config` (stores the default region) and `credentials` (stores access key and secret).
    - You can delete or inactivate these access keys from the `Security credentials` section.

    ```sh
    aws s3 ls

    2020-05-08 11:40:18 johndoe
    ```

    ```sh
    aws s3 ls s3://johndoe

    2020-05-08 11:42:24    3416556 20150217_120527.jpg
    ```

    ```sh
    aws s3 cp s3://johndoe/20150217_120527.jpg 20150217_120527.jpg

    download: s3://johndoe/20150217_120527.jpg to .\20150217_120527.jpg
    ```

    ```sh
    aws s3 mb s3://new-bucket-for-john

    make_bucket: new-bucket-for-john
    ```

    ```sh
    aws s3 rb s3://new-bucket-for-john

    remove_bucket: new-bucket-for-john
    ```

    - You can have CLI commands for all services.
    - If you have `multiple AWS accounts` on your machine:

    ```sh
    aws configure --profile my-other-aws-account
    <Access key>
    <Secret access key>
    <Region>
    <none>
    ```

    ```sh
    aws s3 ls --profile my-other-aws-account
    ```

  - Using AWS CLI on EC2 machine
    - The super bad way is to `aws configure` on an EC2 machine and store credential on it (Others can have access to it and impersonate you.).
    - The best practice is to create a role (on what an instance can do) in IAM Roles and attach it to the EC2 instance. So whenever we use CLI on an instance, AWS will check the permissions of the ROLE.
    - SSH into the instance using the public IP. AWS CLI is already installed in EC2 Linux 2 machines.
    - run `aws configure` but only fill the region (leave out the access key and secret access key).
    - Now, you are not authorized to for example do `aws s3 ls`.
    - Go to IAM -> Roles -> Create Role for EC2 -> Find a managed policy (AmazonS3ReadOnlyAccess) and create the role.
    - Go to EC2 console and right click on the instance to attach the role to the instance.
    - One role can be attached to many EC2 instances (each role can have many policies). But each EC2 instance can have one role at a time.
    - Of course, you can create your own policy and attach it to the role.
    - You can test a policy by `AWS Policy Simulator` tool to choose an action from a service and see if it is allowed or denied.
    - Another way to test a policy without actually running a command (which sometimes can be expensive), is to use `--dry-run` option in front of the CLI command (note that this option is not provided for all commands) to simulate API calls. The result will be either (UnauthorizedOperation with an encrypted failure message [in case of denied] or DryRunOperation [in case of allowed]).
    - You can decode the failure message by using `AWS STS`:
    ```sh
    aws sts decode-authorization-message --encoded-message mxcvnkjbfksjfhksjdfhkasmfn,mn.....
    ```
    - Note that if you want to run the command above from the EC2 instance, you have to add the policy to do the action above from STS service to the role that is attached to this instance.
    - With `curl http://169.254.169.254/latest/meta-data` you can access the `AWS EC2 instance metadata`. If you want to see something specific, you will add it to the URL:
    ```sh
    curl http://169.254.169.254/latest/meta-data/public-ipv4
    ```
    - We don't have to enable anything to access metadata. Each EC2 instance even without an IAM role can see this info about the instance itself.
  - Using AWS SDK on our local computer:
    - What if we want to perform actions on AWS directly from our code? We should use `Software Development Kit`.
    - There are SDKs for each programming languages.
    - Fun fact: AWS CLI uses Python SDK and it is just a wrapper around it.
    - We have to use AWS SDK when coding against AWS services such as DynamoDB.
    - When using SDK, it automatically looks for `.aws` folder in our local machine (in case of IAM user), or instance credentials using IAM role for EC2 instances or from within other AWS services, or environment variables (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY) [the last method is less recommended]. This three ways are `default credential provider chain`.
    - When using SDK, there is exponential back-off for rate limited APIs. Meaning, if a call is failed, the subsequent call will wait twice the time as the previous call.
  - Using AWS SDK on EC2 machine

## ElasticBeanStalk

- Is a developer centric view of deploying an app. It uses EC2, SAG, ELB, RDS, ...
- EB is free but you pay for underlying resources.
- Is a managed service. Although deployment is configurable but performed by EB.
- Three architecture models:
  - Single instance deployment -> good for dev
  - LB + ASG -> great for prod or pre-prod web apps
  - ASG only -> great for non-web apps in production such as workers
- EB has 3 components:
  - Application: each application can have many versions and environments
  - Application version: each deployment gets assigned a version
  - Environment name (dev, test, prod, ...)
- You deploy application versions to environments and can promote application versions to the next environment
- Rollback feature to previous application version.
- You have full control over life-cycle of environments: 1. Create an app and an environment(s) 2. upload a version and give it an alias 3. release to the environments.
- By creating a new web app, giving the application name, choosing the platform (e.g. Node.js), and uploading your code (or using the sample app), it creates a SG, there was an Elastic IP (in dev mode, we get just one instance and with EIP, that can live across instances), an EC2 instance, an S3 bucket, ...
- When you create an app, the first env is dev. Then, we can create another environment and we will be asked what type of env it is: `web server env` or `worker env`. If we need a web app in prod, we choose the first one:
  - give it a name -> MyFirstApp-prod
  - give it a domain
  - choose the platform (e.g Node.js)
  - upload your own code or use sample app.
  - by clicking on `Configure more options` we can have a very finer management over the environment to be created:
    - Choose `High availability` instead of `Low cost` to enable features such as load balancer, auto-scaling,...
    - There are many cards with a `Modify` button under it:
    - Software: we can modify node version, where to store the logs, provide some environments variables, ...
    - Instances: we can modify instance type, instance image id, volume size, ...
    - Capacity: we can modify the number of scaling (e.g. 1 to 4), AZ availability, scaling trigger, ...
    - Load balancer: we can choose application load balancer or classic load balancer and modify each.
    - Rolling updates and deployments (very important, will talk about this in 10 lines down)
    - Security
    - Monitoring
    - Notifications
    - Network
    - Database: we can create an RDS database here but it will be lost if we delete the environment
  - At the end, we click on the `Create environment`. And all the LB, ASG, SGs, EC2s, ... will be created automatically for this prod environment.
    ![](/md/devprod.jpg)
- EB deployment options for updates:
  - `All at once` -> fastest, but instances aren't available to serve traffic for a bit (downtime). This is great for quick iterations in development environment. No additional cost.
  - `Rolling` -> update a few instances at a time (buckets or batches. The bucket size of 2 means to update two instances at a time. You can select a percentage as well.), and the move onto the next bucket once the first bucket is healthy. So at some time, there are two versions of the application. No additional cost. Long deployment.
  - `Rolling with additional batches`-> like rolling but spins up new instances with the new version to move the batch. We are always at capacity. So at some time, there are two versions of the application. Small additional cost. Longer deployment. Good for prod.
  - `Immutable`: spins up new instances with new code in a temporary ASG, deploys version to these instances and then swaps all the instances with the main ASG when everything is healthy. Zero downtime. High cost. Double capacity. Longest deployment. Quick rollback in case of failure. Great for prod.
  - `Blue/Green`: It is more manual and not a direct feature of EB. It has zero downtime. We create a new stage environment and deploy v2 there. The new environment (green) can be validated independently and roll back if issues. Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage env. Using EB `swap URLs` when done with the environment test. So we have DNS change here. It is the longest like above.
- Now, we choose one above, click on the `Upload and Deploy` in the environment and upload the zip file of the updated project.
- All the parameters set in the UI can be configured with code using for example `logging.config` files (which can be in JSON or YML formats) in the `.ebextensions/` directory in the root of the source code. It has also the ability to add resources such as RDS, ElastiCache, DynamoDB, ... Note that, if the environment goes away, all the resources created in this way, will be wiped out as well.
- We can install an additional CLI called the `EB CLI` which makes working with EB from the CLI easier. The commands are like: `eb create`, `eb status`, `eb health`, `eb events`, `eb logs`, `eb open`, `eb deploy`, `eb config`, `eb terminate`, ... It is used to create automated pipeline in CI/CD.
- EB actually under the hood, generates some CloudFormation files for you and CF actually performs the heavy-lifting of updating everything.
- It is better to include dependencies with the source code (in the zip file) to improve deployment performance (otherwise, each EC2 machine needs to resolve dependencies in for example package.json in case of Node.js which is slower).
- HTTPS:
  - We can load the SSL certificate onto the load balancer (either by EB consol>load balancer configuration or from the code: .ebextensions/securelistener-alb.config).
  - That certificate can be provisioned by ACM (AWS Certificate Manager).
  - We must configure a SG rule to allow incoming port 443.
  - We can redirect HTTP to HTTPS by configuring the instance OR configuring the ALB only with a rule.
  - Just make sure not to redirect the health checks (so they keep giving 200 OK).
- EB life-cycle policy:
  - EB can store at most 1000 application versions.
  - If you don't remove old versions, you won't be able to deploy anymore.
  - To phase out old application versions, go to Application versions>Settings and enable life-cyle policy based on time or application version numbers (don't worry, versions that are currently used won't be deleted).
  - There is an option for not to delete the source bundle in S3 to prevent data loss.
- Worker env:
  - If your app performs tasks that are long to complete (for example processing a video, or generating a zip file), offload these tasks to a dedicated worker env. The idea is that by doing this, you decouple your app into two tiers.
  - The web tier (EB: EC2 + ELB) receives HTTP(S) requests and inserts the work into the work queue (SQS), the work queue will be processed by the worker tier (EB: EC2 + SQS) which is a separate new environment.
  - You can define periodic tasks in a file `cron.yml` file. This file will be executed in the worker environment.
- RDS with EB
  - This is great for dev/test but not prod. For prod, it is best to separately create an RDS database and provide the app in EB with the connection string.
  - There are steps to migrate from RDS coupled in EB to standalone RDS, if you have made a mistake and created RDS with EB: Take RDS snapshot -> Enable deletion protection in RDS -> create new env without RDS which points to existing old RDS -> perform blue/green deployment and swap -> terminate the old env (RDS won't be deleted thanks to protection) -> delete CloudFormation stack (which will be in DELETED_FAILED state)

## CICD

- CI: Continuous Integration
  - The developer pushes the code to a repository.
  - The test/build server checks the code as soon as it's pushed.
  - The developer gets feedback about the tests and checks that have passed.
- CD: Continuous Delivery

  - Quick and reliable automated deployment for passed builds (which can be often like 5 releases per day ;)).
  - Tools: CodeDeploy, Jenkins CD, ...

![](/md/cicd.jpg)

### CodeCommit

- To store our code (something like github) and do version control.
- You can have private git repositories.
- No size limit on repositories.
- Fully managed and highly scalable.
- Code only in AWS cloud account -> increased security.
- Secure (encrypted, access control, etc...).
- It can be integrated with Jenkins/CodeBuild/other CI tools.
- Interactions are done using Git. Authentication in Git can be done through SSH keys (upload the SSH keys in the IAM>Users>Click on yourself>Security credentials>SSH keys for AWS CodeCommit) or HTTPS (generate HTTPS git credentials in the IAM>Users>Click on yourself>Security credentials>HTTPS Git credentials) just like github. Then you can clone the repo either by HTTPS or SSH.
- Authorization can be done via IAM policies to manage users/roles rights to repositories.
- Encryption: repos are automatically encrypted at rest using KMS. In transit, we can only use HTTPS or SSH which are secure.
- Cross account access: use IAM role in your account and then the other person has to use AWS STS to call AssumeRole API to have access to your repo.
- Difference with github:
  - security: github has users but CodeCommit has IAM users and roles. github is third party but CodeCommit is within infrastructure.
  - UI: github is winner. CodeCommit is minimal.
- Triggers and Notifications: you can set up triggers and notifications to interact with AWS SNS (Simple Notification Service), AWS Lambda, or AWS CloudWatch Event Rules in case of repository events:
  - Use cases for triggering AWS SNS to for example notify an external build system or AWS Lambda function to for example perform codebase analysis (maybe credentials got committed in the code?):
    - push to existing branch
    - create a branch or tag
    - delete a branch or tag
    - so generally code related events
  - Use cases for notifying CloudWatch Event Rules so that repo users will receive emails about repo events (note that CloudWatch event rule goes into a SNS topic itself. Meaning the target is SNS):
    - pull request updates (create, update, delete, comment)
    - commit comment events
    - so generally around ppl collaborating and chatting around the code

### CodeBuild

- to build and test our code (something like Jenkins CI).
- It is just a server that can run commands for building or testing or both.
- continuous scaling with fully managed build service and there is no build queue.
- You pay for usage: the time it takes to complete the builds.
- Leverages Docker under the hood for reproducible builds.
- Possibility to extend capabilities leveraging our own base Docker images.
- Secure: it has integration with KMS for encryption of build artifacts, IAM for build permissions, VPC for network security, and CloudTrail for API calls loggings.
- Overview:

  - Source code from Github, CodeCommit, S3, ...
  - Build instructions can be defined in code: `buildspec.yml` at the root of the code.

    - Define env variables (for secrets, use SSM Parameter store)
    - There are 4 phases which run in order:

      - install: install dependencies you may need for your build
      - pre_build: final commands to execute before build
      - build: actual build commands
      - post_build: finishing touches (e.g. zip output for example)

        ![](/md/build.jpg)

    - Artifacts: what to upload to S3 (encrypted with KMS)
    - Cache: files to cache (usually dependencies) to S3 for future builds speedup

  - We need a build docker image so we can use either an Amazon-managed Docker image or we could build our own custom image.
  - A CodeBuild container is created and started and will run the commands in the buildspec.yml file.
  - If it passes, the outputs (binaries, zip files,...) will be stored in the S3 bucket of artifacts.
  - Output logs to S3 and/or AWS CloudWatch Logs. After build is finished the whole container that ran the build goes away and the only thing that you have to troubleshoot with is these logs.
  - You can use CloudWatch Alarms to detect failed builds and trigger notifications by defining metrics to monitor CodeBuild statistics.
  - You can use CloudWatch Events and AWS Lambda to glue everything and trigger SNS notifications.
  - Ability to reproduce CodeBuild locally (after installing Docker) to troubleshoot in case of errors.
  - Builds can be defined within CodePipeline (better?) or CodeBuild itself.

### CodePipeline

- Visual workflow to orchestrate and continuous delivery.
- to automate our pipeline from source (we can choose CodeCommit, ECR, S3, or Github and enable CloudWatch Events to detect changes to trigger the pipeline) to build (we can choose CodeBuild or Jenkins) and to finally deploy (we can choose EB, CodeDeploy, ECS, S3, CloudFormation, ...). We choose the application and the environment obviously.
- Made of stages:
  - Each stage can have sequential and/or parallel actions. So an stage can have multiple actions group. Each action group is run after another action group sequentially. Each actions in an action group is run in parallel.
  - Stages example: build, test, deploy, load test, etc.
  - Manual approval can be defined at any stages. The manual reject/approve is accessible via the pipeline itself.
  - Each stage can create outputs artifacts in a S3 bucket and the next stage will get these artifacts.
  - Whenever there is a state change in the pipeline, it will generate an AWS CloudWatch Event and this event can trigger a SNS notification (e.g. you can create events for failed pipelines or cancelled stages).
  - If CodePipeline fails a stage, it stops and you can get info in the console.
  - AWS CloudTrail can be used to audit AWS API calls.
  - If pipeline can't perform an actions, make sure the `IAM Service Role` attached does have enough permissions (IAM Policies).

### CodeDeploy

- to deploy the code to EC2 fleets (not EB).
- not used often.

### CodeStar

- CodeStar is an integrated solution that regroups: github, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch. It brings all together (integrated view). It gives you a nice one-stop dashboard.
- Helps quickly create `CICD-ready` projects fro EC2, Lambda, EB.
- Limited customization. Just simple and quick to start.

## CloudFormation

- Managing your infrastructure as code.
- Currently, we've been doing a lot of manual work which will be very tough to reproduce.
  - in another region
  - in another AWS account
  - within the same region if everything was deleted
- wouldn't be great, if all our infrastructure was ... code?
- That code would be deployed and create/update/delete our infrastructure, instead of clicking here and there.
- CF is a declarative way of outlining your AWS infrastructure (for most of services).
  - I want an SG.
  - I want two EC2 using this SG.
  - I want two Elastic IPs for these EC2s.
  - I want an S3 bucket.
  - I want an ELB in front of these.
- CF will create these for you in the right order, with the exact configuration.
- Benefits:
  - Infrastructure as code which can be version controlled using git and changes can be reviewed.
  - Cost: Each stack that you create (so you will create stack) has an ID and you can track the cost of each stack (Each stack has one template and many resources. An example of a stack is an environment in EB.). You can estimate the costs of your resources using CF template. In dev, you can automate the deletion of templates at 6 pm and recreate them at 8 am.
  - Productivity: Ability to destroy and recreate an infrastructure on the cloud on the fly. Because of declarative programming, there is no need to figure out ordering and orchestration. You can also view and edit the template in the Designer which can be helpful for presentation.
  - Separation of concerns: you can create many stacks for many apps, and many layers: e.g. VPC stacks, network stacks, app stacks.
  - Don't reinvent the wheels: leverage existing templates on the web and the huge documentation.
- How it works?
  - Templates have to be uploaded in S3 and then referenced in CF.
  - To update a stack (Actions>Update Stack), we can't edit the previous one. We have to re-upload a new version of the template to AWS. CF will compare it with the older version of the stack and will add or modify resources.
  - Stacks are identified by a name.
  - Delete a stack deletes all artifacts and resources that were created by CF in the right order.
- Manual way: Editing templates in the CF Designer and using the console to input parameters, etc.
- Automatic way: Editing templates in a YAML file and using AWS CLI to deploy templates. This way is recommended to fully automate your flow.
- YML: Key-value pairs. Nested objects. Support arrays.

```yml
invoice: 1
date: 2001-01-23
bill-to:
  name: ben
  family: abe
  address: |
    unit 111, abc street.
    suit 1
  city: NY
products:
  - id: 1
    quantity: 2
    price: 23
  - id: 2
    quantity: 1
    price: 12
```

- Templates components:

  - Resources: your AWS resources (EC2, EIP, SG, LB, ...) declared in the template (mandatory).

    - Resources can reference each other and be linked together.
    - AWS figures out creation, updates and deletes of resources for us.
    - There are 224 types of resources.
    - You cannot create dynamic amount of resources. Everything in the CF template has to be declared. You can't perform code generation there.
    - Resource types identifiers are of the form of `AWS::aws-product-name::data-type-name`. For example:

    ```yml
    Parameters:
      SecurityGroupDescription:
        Description: Security Group Description
        Type: String

    Resources:
      MyInstance:
        Type: AWS::EC2::Instance
        Properties:
          AvailabilityZone: us-east-1a
          ImageId: ami-a4c7edb2
          InstanceType: t2.micro
          SecurityGroups:
            - !Ref SSHSecurityGroup
            - !Ref ServerSecurityGroup
      MyEIP:
        Type: AWS::EC2::EIP
        Properties:
          InstanceId: !Ref MyInstance
      SSHSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: Enable SSH access via port 22
          SecurityGroupIngress:
            - CidrIp: 0.0.0.0/0
              FromPort: 22
              IpProtocol: tcp
              ToPort: 22
      ServerSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: !Ref SecurityGroupDescription
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 80
              ToPort: 80
              CidrIp: 0.0.0.0/0
            - IpProtocol: tcp
              FromPort: 22
              ToPort: 22
              CidrIp: 192.168.1.1/32
    ```

  - Parameters: the dynamic inputs for your template.
    - If you want to reuse templates across the company.
    - Or if some inputs can not be determined ahead of time. For example the key pair you want to link to your EC2 instances.
    - Parameters have types so they can prevent error.
    - Parameters settings:
      - Types: String, Number, CommaDelimitedList, List<Type>, AWS Parameter
      - Description
      - Constraints: Min/MaxLength, Min/MaxValue, Defaults, AllowedValues (array), AllowedPattern (regexp), NoEcho (Boolean for secrets), ConstraintDescription (String)
    - If you use parameters, you won't have to re-upload another template to change just one variable.
    - `!Ref` or `Fn::Ref` function can be used to reference a parameter or a resource.
    - Parameters can be used anywhere in a template.
    - Pseudo Parameters: these kind of parameters are enabled by default and can be used at any time and I guess they refer to the context in which the template is running:
      - AWS::AccountId -> 1234567890
      - AWS::Region -> us-east-2
      - AWS::StackId -> arn:aws:cloudformation:us-east-1:123456789012:stack/MyStack/1c2fa620-9B2a-54asd24g5r12
      - AWS::StackName -> MyStack
      - AWS::NoValue -> does not return a value
      - AWS::NotificationARNs -> [arn:aws:sns:us-east-1:123456789012:MyTopic]
  - Mappings: the static variables for your template.
    - They are fixed variables within the CF template.
    - They are very handy to differentiate between different environments (dev vs prod), regions, AMI types, etc.
    - All the values are hard-coded within the template.
    ```yml
    Mappings:
      RegionMap:
        us-east-1:
          "32": "ami-6411e20d"
          "64": "ami-7a11e213"
        us-west-1:
          "32": "ami-c9c7978c"
          "64": "ami-cfc7978a"
    ```
    - Mappings are great when you know in advance, all the values that can be taken and that they can be deduced from variables such as Region, AZ, AWS Account, Environment, etc.
    - They are safer than parameters.
    - We can access them like: `!FindInMap [RegionMap, !Ref "AWS::Region", 32]`
  - Outputs: we can export from our template and other templates can reference it.
    - They are very useful for example if you define a network CF and output the variables such as VPC ID and your Subnet IDs.
    - It enables cross stack collaboration, as you let expert handle their own part of the stack.
    - You can't delete a CF stack if its outputs are being referenced by another CF stack.

  ```yml
  Outputs:
    StackSSHSecurityGroup:
      Description: The SSH Security for our company
      Value: !Ref MyCompanyWideSSHSecurityGroup # this is within the resources
      Export:
        Name: SSHSecurityGroup
  ```

  ```yml
  Resources:
    MyInstance:
      Type: AWS::EC2::Instance
      Properties:
        AvailabilityZone: us-east-1a
        ImageId: ami-a4c7edb2
        InstanceType: t2.micro
        SecurityGroups:
          - !ImportValue SSHSecurityGroup
  ```

  - Conditionals: if-statements to perform resource creation based on a condition.

    - Conditions can be anything but common ones are environment, region, any parameter value.
    - Each condition can reference another condition, parameter value or mapping.
    - Logical functions: `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, `Fn::Or`.
    - Conditions can be applied to resources, outputs, etc.

    ```yml
    Conditions:
      CreateProdResources: !Equals [!Ref EnvType, prod] #EnvType is maybe a parameter for example
    ```

    ```yml
    Resources:
      MountPoint:
        Type: AWS::EC2::VolumeAttachment
        Condition: CreateProdResources
    ```

  - Functions: to transform data within your templates.
    - Fn::Ref -> returns the value of a parameter or the physical ID of the underlying resource if used with resources
    - Fn::GetAtt -> to return other attributes of a resource (e.g. EC2Instance) other than ID from above: `!GetAtt EC2Instance.AvailabilityZone`
    - Fn::FindInMap
    - Fn::ImportValue
    - Fn::Join -> `!Join [":", [a,b,c]]` -> "a:b:c"
    - Fn::Sub -> is used to substitute variables from a text.
    - Condition functions: Fn::And, Fn::Equals, Fn::If, Fn::Not, Fn::Or.

- CF Rollbacks:
  - Stack creation fails:
    - Default: everything rolls back (gets deleted) and we can look at the logs.
    - Option to disable rollback and troubleshoot what happened.
  - Stack update fails:
    - The stack automatically rolls back to the previous known working state.
    - Ability to see in the logs what happened and error messages.

## Monitoring

- Monitoring is important because the users only care that the application is working with a high quality experience. Also, we can prevent issues before they happen or adjust scaling patterns to use resources effectively.

### CloudWatch

#### Metrics

- Collects and tracks key metrics.
- CW provides metrics for every services in AWS.
- Metric is a variable to monitor (CPU utilization, Network, ...).
- Metrics are grouped by namespaces (EC2 has 357 metrics, S3 has 8 metrics, ...). By clicking on each you have further categories for example by clicking on EC2, we have by ASG, per-instance metrics, across all instances, ...
- Dimension of a metric is an attribute of it like instance id in CPU utilization metric. We can have up to 10 dimensions per metric.
- Metrics have timestamp (when they are sent to CW).
- We can add metrics to CW dashboards (a graph widget in the root of CW for for example CPU utilization of a ASG).
- EC2 instance metrics have metrics every 5 mins, with a cost, we can have EC2 detailed monitoring and you can get data every 1 minute (10 detailed monitoring metrics for free tier). Use detailed monitoring if you want to scale ASG more quickly.
- Note that EC2 memory usage is not pushed by default and it must be pushed from inside the instance as a custom metric.
- Custom metrics:
  - Ability to use dimensions (attributes) to segment metrics such as instance id or environment name.
  - The standard metric resolution is 1 minute but you can have high resolution up to 1 second by calling StorageResolution API with a higher cost.
  - To send a metric to CW, you have to call PutMetricData and use exponential back off in case of throttle errors.

#### Alarms

- Reacts in real-time to metrics/events.
- Alarms are used to trigger notifications for any metric (we define a threshold).
- Alarms can be attached to ASGs, EC2 actions, or SNS notifications.
- You can customize alarms based on various options (sampling, %, average, max, min, etc).
- Period is the length of time in seconds to evaluate the metric. High resolution metrics can only choose 10 secs or 30 secs.
- Alarms can be in different states:
  - OK
  - INSUFFICIENT_DATA
  - ALARM
- Alarms actions (which can be related to ASGs, EC2 actions, or SNS notifications) can be defined based on the state of the alarm.

#### Logs

- Collects, monitors, analyzes and stores log files.
- CW can collect logs from EB, ECS, AWS Lambda, VPC, API Gateway, CloudTrail, CloudWatch, Route 53, ...
- CW logs can go to S3 for archival or ElasticSearch cluster for further analytics.
- CW logs can use filter expressions.
- Logs storage: logs groups (usually representing an application) and log streams (usually instances within an application or for example any build of a CodeBuild).
- You can define log expiration policies (never expires, 30 days, etc.)
- With AWS CLI we can show CW logs.
- To send logs to CW, make sure that IAM permissions are correct.
- Logs can be encrypted using KMS at the group level.

#### Events

- Sends notifications when certain events happen in your AWS.
- You have to create an event rule (source of the event (can be scheduled (cron jobs) or has a pattern when a service is doing something like CodePipeline state changes) and the target (Lambda function, SQS/SNS/Kinesis)).
- CW Event creates a small JSON document to give info about the change.

### X-Ray

- Allows you to troubleshoot your application performance and errors.
- Distributed tracing (end to end way to follow a request) of microservices. You can trace calls between different components and how they interact with each other.

### CloudTrail

- Internal monitoring of API calls being made.
- Audit changes to AWS resources by your users.

## Integration and Messaging

- When we start deploying multiple applications, they will need to communicate with one another:
- There are two types of communications:
  - Synchronous (app to app) -> direct call of the API.
  - Asynchronous or event-based (app to queue and another app from queue) -> indirectly
- Synchronous communication can be problematic if there are sudden spikes of traffic. For example, 1000 request for video encoding. In this case, it is better to `decouple` your app (live user requests from intensive background works) so that these services can scale independently from each other (users can upload media while encoding it):
  - Queue model -> SQS
  - Pub/sub model -> SNS
  - real-time streaming or big data analytics -> Kinesis

### Simple Queue Service: SQS

- Producer(s) can send messages to the SQS queue and consumer(s) can poll messages from it.

#### Standard Queue

- It is the oldest service and fully managed.
- It can scale from 1 message per second to 10,000 per second.
- Default retention is 4 days and can be extended up to 14 days.
- Once you consume a message, it is gone.
- No limit on how many messages can be in the queue.
- Low latency (<10 ms on publish and receive>).
- Horizontal scaling in terms of number of consumers (we can have as many consumers as we want).
- Can have duplicate messages (this is rare). Sometimes the producer sends twice the message by design. A message is delivered at least once. But occasionally more than one copy of a message is delivered.
- You can have out of order messages (best effort ordering). Sometimes SQS delivers the messages in a different order than when it has received by design.
- Limitation of 265KB per message sent (So send data and JSON not video!).

#### Delay Queue

- We can delay a message (consumers don't see it immediately) up to 15 mins.
- Default is 0 for standard queue (message is available right away).
- We can set a default at queue level.
- We can override the default using the DelaySeconds parameter when we send data to SQS.

#### Messages

- We define a body up to 256KB which is a string.
- We can add message attributes which are metadata and optional -> key-value pairs: Name, Type, Value
- We also can provide delay delivery as seen above.
- When we send it to SQS, we get a message identifier and the MD5 hash of the body (if you want to store it for any reason).
- Consumers (workers) polls SQS for messages and can receive (the better verb is viewing) up to 10 messages at a time.
- Consumers (workers) can process the message within the visibility timeout and then ask SQS to delete the message using the message Id and receipt handle (receipt handle has been given when we view a message). We have to provide receipt handle when sending delete request for a message.
- SQS is only for transporting not storing.

#### Visibility timeout

- When a consumer polls a message from a queue, the message is `invisible` to other consumers for a defined period of `visibility timeout` (default is 30 seconds but can be 0 seconds up to 12 hours).
- If it is set too high (15 mins) and consumer fails to process the message, you must wait a long time before processing the message again.
- If it is set too low (30 seconds) and consumer needs more time to process it like 2 mins, the message will be visible to other consumers and there will be a chance that a message be processed multiple times.
- There is a `ChangeMessageVisibility` API to change the visibility while processing it to extend the timeout period.
- There is a `DeleteMessage` API to to tell SQS that the message was successfully processed.

#### Dead letter queue

- If a consumer fails to process a message within visibility timeout, the message goes back to the queue.
- We can set a threshold of how many times a message can go back to the queue, it is called a `redrive policy`.
- After the threshold is exceeded, the message goes into a dead letter queue (DLQ).
- We have to create DLQ first and then designate it DLQ.
- Make sure to process the messages in the DLQ before they expire.

#### Long polling

- When a consumer requests message from the queue, it can optionally wait (1 to 20 seconds, 20 seconds is preferable) for messages to arrive if there are none in the queue -> long polling.
- Long polling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application.
- Long polling can be enabled at the queue level or at the API level using `WaitTimeSeconds` so the consumer tells that I can wait this much.

#### SQS CLI

- You can create and configure a SQS using console and can send/view/delete messages out of it.
- You can use AWS CLI as well (if the region that the SQS is in is different from the default region) to make API calls:

```sh
aws sqs list-queues --region us-east-1
```

```sh
aws sqs send-message --queue-url https://queue.amazonaws.com/38723425345/MyFirstQueue --region us-east-1 --message-body hello-world
```

```sh
aws sqs receive-message --queue-url https://queue.amazonaws.com/38723425345/MyFirstQueue --region us-east-1 --visibility-timeout 30 --wait-time-seconds 20
```

```sh
aws sqs delete-message --receipt-handle AJHGJHGJ+asfkjsJHGJYBIUBYT%&RYGfhgdtrvrUTYRVUVRUGKJHjkhkugjfHFYT --queue-url https://queue.amazonaws.com/38723425345/MyFirstQueue --region us-east-1
```

#### FIFO Queue

- First In First Out -> not available in all regions.
- Name of the queue must end in .fifo.
- It has lower throughput (up to 3,000 per second with batching, 300/s without).
- Messages are processed in order by the consumer.
- Messages are sent exactly once.
- There is no per message delay (only per queue delay. otherwise, it can not gurantee FIFO.).
- It is guaranteed for ordering vs best effort.
- `Deduplication`
  - There are two types of deduplication. For one, you have to provide MessageDeduplicationId with your message and deduplication interval is 5 minutes.
  - You can have content based deduplication -> the MessageDeduplicationId is generated as the SHA-256 of the message body (not the attributes).
- `Sequencing`:
  - To ensure strict ordering between messages, specify a MessageGroupId.
  - Messages with the same group id are delivered to one consumer at a time. For example, to order messages for a user, you could use the user_id as a group id.
  - Messages with the different group id may be received out of order or consumed by different consumers at a time.

#### SQS Extended Client

- Max size limit is 256Kb, to send larger messages, we can use the SQS extended client (Java library).-
- It sends large message to S3 and small metadata to SQS queue. The consumer receives the small metadata from the queue and then large message from S3.

#### Other

- Encryption in flight using theHTTPS endpoint.
- IAM policy must allow usage of SQS.
- SQS queue access policy for finer control over which IP can access SQS or control over the time the requests come in.
- There is no VPC endpoint so it must be through internet to access SQS.
- APIs:
  - CreateQueue, DeleteQueue
  - PurgeQueue to delete all the messages in queue
  - SendMessage, ReceiveMessage, DeleteMessage
  - ChangeMessageVisibility to change the timeout
  - BatchSendMessage, BatchDeleteMessage, BatchChangeMessageVisibility to decrease your costs

### Simple Notification Service: SNS

- When you want to send one message to many receivers.
- A service (event producer) publishes one message to SNS topic (topic is a message channel) and subscribers or event receivers (up to 10,000,000 per topic) will be notified.
- Each subscriber to the topic will get all the messages.
- You can have up to 100,000 topics.
- Data is not persisted (lost if not delivered).
- Subscribers can be
  - SQS queue
  - HTTP/HTTPS endpoints (with delivery retries- how many times)
  - Lambda (very common subscriber)
  - Emails
  - SMS messages
  - Mobile Notifications
- SNS can be integrated with many Amazon services:
  - CloudWatch (for alarms or event rules)
  - ASG notifications
  - S3 (on bucket events)
  - CloudFormation (upon state changes such as failed to build)
  - etc
- How to publish
  - We can use SDK to create a topic.
  - Create one or many subscriptions
  - Publish to the topic
- SNS + SQS: Fan Out
  - One service publishes a message to SNS topic and two or more SQS queues are registered to process that message.
  - Push once in SNS and receive in many SQS.
  - Fully decoupled.
  - No data loss (if we had only SNS, if the other services were down we would have data loss)
  - Ability to add receivers of data later
  - SQS allows for delayed processing
  - SQS allows for retries of work
  - May have many workers on one queue and one worker on the other queue

### Kinesis

- a managed alternative to Apache Kafka.
- It's a big data streaming tool which allows you to collect application logs, metrics, IoT, clickstreams basically anything that is real-time big data.
- It's compatible with many streaming processing frameworks such as Apache Spark, Nifi, etc (These are frameworks that helps you perform computations in real time on data that arrives through a stream).
- Data is automatically replicated to 3 AZ.
- There are three sub kinesis products:

  - Kinesis Streams: to ingest (absorb) streams at scale with a low latency

    - Streams are divided in ordered shards
    - each shard is like a queue.
    - To increase the throughput we can add shard.
    - Data retention is 1 day by default and can go up to 7 days.
    - Remember that Kinesis is a massive highway and you want to process data and do something with it and put it somewhere else as soon as possible. Kinesis is just a tool to allow you have that throughput in real-time.
    - Ability to reprocess and replay data. In SQS, once the data is processed it's gone but here it will expire after some time.
    - Multiple applications can consume the same stream (something like SNS).
    - Real-time processing with scale of throughput.
    - Once data is inserted, it can't be deleted (immutability). add data -> process by consumers -> data stays 1 to 7 days and you do something with it.
    - Shards:
      - 1MB/s or 1000 messages/s at write per shard
      - 2MB/s at read per shard
      - billing is per shard provisioned so select shards to be fully utilized.
      - The number of shards can evolve over time.
      - Records are ordered per shard.
    - With PutRecord API, you can send data to Kinesis. For this, you have to send data and a partition key.
    - Message key is whatever you want (a string) but you have to choose one that is well distributed because all the data with one message key (this key gets hashed to determine the shard id) goes into the same shard (to help with ordering for a specific key. The message sent to a shard gets a sequence number.) and it can be overwhelmed (hot partition) so user id is better than country id.
    - If we go over the limits of a shard, we get ProvisionedThroughputExceeded. Solution: retries with back-off, increase shards (scaling), ensure your partition key is a good one.
    - You can use CLI or SDK to produce and send messages.
    - Practice:
      - First create the stream in the console with one shard.

    ```sh
    aws kinesis list-streams

    {
      "StreamNames": [
        "my-first-stream"
      ]
    }
    ```

    ```sh
    aws kinesis put-record --stream-name my-first-stream --data "user signed up" --partition-key user_123

    {
      "ShardId": "shardId-00000000000",
      "SequenceNumber": "49654532132132132135454876876543258465879876514621",
    }
    ```

    ```sh
    aws kinesis get-shard-iterator --stream-name my-first-stream --shard-id shardId-00000000000 --shard-iterator-type TRIM_HORIZON

    {
      "ShardIterator": "AAAAAAAAAAAAADflkjhkjhKJGJHkjbilugnuyg",
    }
    ```

    ```sh
    aws kinesis get-records --shard-iterator "AAAAAAAAAAAAADflkjhkjhKJGJHkjbilugnuyg"

    {
      "Records": [
        {
          "SequenceNumber": "49654532132132132135454876876543258465879876514621",
          "ApproximateArrivalTimestamp": 15365461621.562,
          "Data" : "dXNSDGFDFGsddfsdf=",
          "PartitionKey": "user_123"
        }
      ]
    }
    ```

    - The data is base64 encoded.

  - Kinesis Analytics: to perform real-time analytics (filters, computations, aggregations) on streams using SQL
    - Auto scaling
    - Managed: no servers to provision
    - Continuous: real time
    - Pay for actual consumption rate
    - Can create streams out of the real-time queries
  - Kinesis Firehose: to load streams into S3, ElasticSearch, ...
    - Fully managed service, no adminstration
    - Near real time (60 seconds latency)
    - load data into S3/Redshift/ElasticSearch/Splunk
    - Auto scaling
    - Pay for the amount of data going through Firehose

![](/md/kinesis.jpg)

# Serverless

- Is a new paradigm in which developers don't have to manage servers anymore.
- They just deploy code (functions).
- Initially it was FaaS (Function as a service).
- Serverless was pioneered by AWS Lambda but also includes anything tha is managed: "databases, messaging, storage, etc".
- Serverless does not mean that there are no servers. It means you just don't manage/provision/see them.
- Apart from Lambda, DynamoDB, Cognito, and API Gateway, S3, SNS, SQS, Kinesis, and Aurora are serverless services.

![](/md/serverless.jpg)

## Lambda

- EC2 is
  - Virtual servers in the cloud.
  - Limited by RAM and CPU.
  - Continuously running.
  - Scaling means intervention to add/remove servers.
- Lambda is
  - Virtual functions so no server to manage.
  - Limited by time -> short execution.
  - Run on-demand.
  - Scaling is automated.
- Benefits of Lambda:
  - Easy and cheap pricing: pay per request and compute time: First 1,000,000 requests are free. 400,000 GB-seconds of compute time per month is free meaning you can have 400,000 seconds if function is 1GB RAM.
  - Integrated with the whole AWS stack: API Gateway, Kinesis, DynamoDB, S3, IoT, CW Events, CW Logs, SNS, Cognito, SQS, CodeCommit.
  - Easy monitoring through CW.
  - Easy to get more resources per function (up to 3GB of RAM).
  - Increasing RAM will also improve CPU and network.
- Example:
  - Upload an image in S3.
  - It triggers a Lambda function to create a thumbnail for it.
  - Lambda pushes the generated thumbnail to the S3 and the metadata to DynamoDB.
- Another example:

  - CloudWatch Events emits events based on the CRON schedule.
  - It triggers a Lambda function to perform a task.
    Lambda function:
  - when creating a Lambda function, you have to select the IAM role, for example `Create a new role with basic Lambda permissions` which is the permission to upload logs to Amazon CloudWatch Logs.

- The Lambda function code is something like:

  ```js
  console.log("Loading function");

  exports.handler = async (event, context) => {
    //console.log('Received event:', JSON.stringify(event, null, 2));
    console.log("value1 =", event.key1);
    console.log("value2 =", event.key2);
    console.log("value3 =", event.key3);
    return event.key1; // Echo back the first key value
    // throw new Error('Something went wrong');
  };
  ```

  - You can test the function by creating a new test event like:

  ```json
  {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  }
  ```

  - If you see the execution result:

  ```dos
  The area below shows the result returned by your function execution. Learn more about returning results from your function.

  "value1"

  Log output
  The section below shows the logging calls in your code. These correspond to a single row within the CloudWatch log group corresponding to this Lambda function. Click here to view the CloudWatch log group.

  START RequestId: e148ce21-2886-4ff8-926f-e887741e1167 Version: \$LATEST
  2020-05-14T02:43:26.751Z e148ce21-2886-4ff8-926f-e887741e1167 INFO value1 = value1
  2020-05-14T02:43:26.751Z e148ce21-2886-4ff8-926f-e887741e1167 INFO value2 = value2
  2020-05-14T02:43:26.751Z e148ce21-2886-4ff8-926f-e887741e1167 INFO value3 = value3
  END RequestId: e148ce21-2886-4ff8-926f-e887741e1167
  REPORT RequestId: e148ce21-2886-4ff8-926f-e887741e1167 Duration: 17.60 ms Billed Duration: 100 ms Memory Size: 128 MB Max Memory Used: 63 MB Init Duration: 125.57 ms
  ```

  - If we uncomment `// throw new Error('Something went wrong');` and test it again:

  ```json
  {
    "errorType": "Error",
    "errorMessage": "Something went wrong",
    "trace": [
      "Error: Something went wrong",
      "    at Runtime.exports.handler (/var/task/index.js:9:12)",
      "    at Runtime.handleOnce (/var/runtime/Runtime.js:66:25)"
    ]
  }
  ```

- Configuration:
  - Timeout: default is 3 seconds, max 900 seconds.
  - Environment variables
  - Allocated memory (128MB to 3 GB)
  - Ability to deploy within a VPC + assign an SG -> so we can talk to RDS instance in the VPC by deploying Lambda in the same VPC. The default is no VPC. If you select a VPC, you have to choose a subnet and a SG as well.
  - IAM execution role with correct permissions must be attached to the Lambda function to allow it do whatever it supposes to do.
- You can edit the code in the console or upload a zip file.
- The handler is the function in the code that the Lambda will call it when it is triggered.
- Concurrency and Throttling:
  - Up to 1000 concurrent executions.
  - We can limit this by specifying `Reserve concurrency` at the function level.
  - Each invocation over the concurrency limit will trigger a `throttle`.
  - Throttle behavior:
    - Synchronous invocation (like API Gateway or console) => returns ThrottleError- 429
    - Asynchronous invocation (like S3) => retry automatically and then go to DLQ.
    - DLQ can be a SNS topic (an email) or SQS queue (the message body is the original event payload).
    - This is an easy way to debug what's wrong with your functions in production without changing the code.
    - Make sure that the IAM execution role is correct for your Lambda function.
- Logging, Monitoring and Tracing
  - CloudWatch:
    - AWS Lambda execution logs are stored in AWS CloudWatch logs.
    - AWS Lambda metrics are displayed in AWS CloudWatch metrics.
    - Make sure that AWS Lambda function has an execution role with an IAM policy that authorizes writes to CloudWatch.
  - X-Ray:
    - It's possible to trace Lambda with X-ray.
    - Enable in Lambda configuration. It runs the X-Ray daemon for you.
    - Use AWS SDK in code.
    - Ensure Lambda function has correct IAM execution role.
- Limits
  - Disk capacity in the function container is 512 MB in `/tmp` folder.
  - Lambda function deployment size: compressed .zip -> 50MB, uncompressed (code + dependencies) -> 250MB
  - You can use the `/tmp` folder to load other files at startup.
  - Size of environment variables -> 4KB
- Versions
  - When we work on a Lambda function, we work on the \$LATEST (mutable).
  - When we are ready to publish a Lambda function, we create a version.
  - Versions are immutable (snapshots of \$LATEST).
  - Versions have increasing version numbers.
  - Versions get their own ARN.
  - Version = code + configuration (immutable -> nothing can be changed).
  - Each version of the Lambda function can be accessed using the correct ARN.
  - We create versions to have a dev, test, and prod environments. In this way, we have to rewire in different environments every time we have a new version -> painful -> So we create Lambda Aliases.
  - Aliases are pointers to Lambda function versions.
  - We can define a dev, test, and prod aliases and have them point at different lambda versions.
  - Aliases are mutable.
  - Aliases enable Blue/Green deployment by assigning weights to lambda functions. Users don't see it, because they are interacting with aliases and nothing is changed and endpoints are the same.
  - Aliases have their own ARNs.

![](/md/aliases.jpg)

- Dependencies

  - If your Lambda function depends on external libraries (AWS SDK, DB clients, etc.), you need to install the packages alongside your code and zip it together. For example for Node.js, use npm and `node_modules` directory.
  - Upload the zip straight to Lambda if it is less than 50MB, else to S3 first.

  ```js
  //Require the X-Ray SDK (need to install it first)
  const AWSXRay = require("aws-xray-sdk-core");

  //Require the AWS SDK (comes with every Lambda function so no need to install)
  const AWS = AWSXRay.captureAWS(require("aws-sdk"));

  //We'll use the S3 service, so we need a proper IAM role
  const s3 = new AWS.S3();

  exports.handler = async function (event) {
    return s3.listBuckets().promise();
  };
  ```

- Lambda and CloudFormation
  - You must store the Lambda zip in S3 in the same region that you want to execute the Lambda function.
  - You must refer the S3 zip location in the CloudFormation template.
    - We can have `S3BucketParam` and `S3KeyParam` as parameters to point to where the code is (bucket name and the path).
    - We can have two resources: `LambdaExecutionRole` (IAM role with policies to logs, s3, and xray) and `LambdaWithXRay` (Which is an AWS Lambda function). The code is referenced to the parameters in LambdaWithXRay resource.
  - Create a new stack in the CloudFormation, upload the template, and enter the parameters (the bucket that has the code and the key to the code).
- Best Practices
  - Perform heavy-duty work outside of your handler function. Just outside function body, still inside code.
    - Connect to database outside of your handler function.
    - Initialize the AWS SDK outside of your handler function.
    - Pull in dependencies or datasets outside of your handler function.
  - Use environment variables for
    - Database connection strings, S3 bucket, etc. Do not put these values directly in the code.
    - Passwords, sensitive values, ... can be encrypted using KMS.
  - Minimize your deployment package size to its runtime necessities.
    - Break down the function if it is needed.
    - Remember the AWS Lambda limits.
  - Avoid using recursive code, never have a Lambda function call itself.
  - Don't put your Lambda function in a VPC unless you have to (It increases latency).

## DynamoDB

- NoSQL serverless database.
- NoSQL databases do not support join (They expect you to have all the info you need within one table or to do the join client side).
- NoSQL databases don't perform aggregation such as SUM.
- NoSQL databases are distributed and scale horizontally to respond to millions of requests per second (scale by adding more machine. In contrast to vertical scaling which means adding more power).
- Fully-managed. Highly available with replication across 3 AZ (distributed).
- Fast and consistent in performance (low latency on retrieval).
- Integrated with IAM for security, authorization, and administration.
- Enables event driven programming with DynamoDB streams.
- Is made of `tables`.
- Each table has a primary key (which must be decided at creation time).
  - Option 1: Partition key only (HASH): it must be unique for each item and must be diverse enough so that the data is distributed across many partitions. -> for example user_id (random series of chars) for a users table.
  - Option 2: Partition key + Sort key (=range key) (the combination must be unique). Data is grouped together by partition key. For example user_id (partition key) plus game_id (sort key) where user can play many games and the result is the attribute. The partition key is better to be chosen the attribute with highest cardinality. This way the data will be distributed better. For example if we want to create a movie table, the movie_id is better choice for partition key in comparison to movie_language.
- Each table can have infinite number of `items` (=rows).
- Each item has `attributes` (can be null or nested so it is more than columns). Items are schema-less.
- Maximum size of an item is 400KB.
- Data types supported are
  - String, Number, Binary, Boolean, Null
  - Document types: List and Map
  - Set types: String Set, Number Set, Binary Set
- The database is there, you just have to create tables.
- Only the partition and sort keys must not be null. All other attributes can be null.
- Each item can have different attributes than other attribute.
- On-demand mode:
  - It is more flexible for new tables that we can't predict the read and write capacity.
  - It is PAYG.
- Provisioned throughput:
  - Allow you to reserve capacity in advance, reducing the cost significantly.
  - `Tables` must have provisioned with read and write capacity units (RCU and WCU) which are throughput. So remember they are provisioned for each table.
  - One WCU represents one write per second for an item up to 1KB in size. It rounds up, so to write 1 object of size 1.1 KB in 1 sec, 2 WCUs are needed.
  - One RCU represents one strongly-consistent or two eventually-consistent read per second for an item up to 4KB in size. It rounds up too.
  - So if application is more read-heavy, have more RCUs or vice versa.
  - There is an option to setup auto scaling of throughput to meet the demand.
  - Throughput can be exceeded temporarily using burst credit.
  - If burst credit is empty, you will get a `ProvisionedThroughputException`.
  - It's then advised to an exponential back-off retry.
- `Eventually-consistent read` means if we read just after a write, it is possible to get an unexpected response because of replication.
- `Strongly-consistent read` means if we read just after a write, we will get the correct data. It has a few hundreds milliseconds latency.
- By default, DynamoDB uses eventually-consistent reads but GetItem, Query, and Scan provide a `ConsistentRead` parameter you can set to true.

![](/md/eventually.jpg)

- Partitions:
  - Data is divided in partitions.
  - Partition keys go through a hashing algorithm to know which partition they go to (That is why we have to choose partition keys to be of higher cardinality -> otherwise most of data will go into one partition and because WCU and RCU are spread evenly between partitions, the performance would drop).
  - The number of partitions = CEILING(MAX(capacity = RCU/3000 + WCU/1000, size = Total size/10GB)).
  - If we exceed our RCU or WCU, we get ProvisionedThroughputException.
    - Reasons: Hot keys (one partition key is being read too many times, popular item for example), hot partitions (one partition is being read too many times because of poor choice of partition key), or a very large item (a 400KB item).
    - Solutions:
      - Exponential back-off when exception is encountered (already in SDK).
      - Distributed partition keys as much as possible.
      - If RCU issue, we can use DynamoDB Accelerator (DAX) which is basically a cache.
- APIs:
  - PutItem: Write data to DynamoDB (create data or full replace).
  - UpdateItem: Update data in DynamoDB (partial update of attributes)
  - ConditionalWrites: Accept a write/update only if conditions are respected, otherwise reject. It helps with concurrent access to items. No performance impact. That makes DynamoDB an optimistic locking/concurrency database.
  - DeleteItem: Delete an individual row and it has ability to perform a conditional delete.
  - DeleteTable: Delete a whole table and all its items. Much quicker than deleting all items using DeleteItem.
  - BatchWriteItem: Up to 25 PutItem and/or DeleteItem in one call. Up to 16 MB of data written (400KB per item). Batching allows you to save in latency by reducing number of API calls done against DynamoDB. Operations are done in parallel for better efficiency. It is possible for part of a batch to fail in which it's up to you to retry the failed items (using exponential back-off algorithm).
  - GetItem: Read based on primary key (HASH or HASH-RANGE). Eventually consistent read by default. Operation to use strongly consistent reads (more RCU might take longer). `ProjectionExpression` can be specified to include only certain attributes.
  - Batch GetItem: Up to 100 items. Up to 16MB of data. Items are retrieved in parallel to minimize latency.
  - Query: returns items up to 1MB or number of items specified in the limit (it can skip item for pagination too) based on
    - PartitionKey value (must be = operator).
    - SortKey (=, <, <=, >, >=, between, Begin) -> optional.
    - FilterExpression to further filter (client side filtering).
  - Scan: Scans the entire table and filters out data (extremely inefficient and consumes a lot of RCU). For faster performance, use parallel scans (multiple instances scan multiple partitions at the same time at the expense of RCU). You can use ProjectExpression + FilterExpression with scan too.
- Local Secondary Index (LSI)
  - An index that has the same partition key as the table, but a different sort key.
  - Alternate range key for your table, it is local to the partition key.
  - Up to five local secondary indexes per table.
  - The key should consist of exactly one scalar attribute (String, Number, or Binary).
  - LSI must be defined at table creation time.
  - LSI uses the WCU and RCU of the main table so no special consideration for throttling.
- Global Secondary Index (GSI)
  - To speed up queries on non-key attributes, use a GSI.
  - GSI = a new partition key + optional sort key -> This index can be seen as a whole new table and we can project other attributes on it (the partition key and sort key of the original table are always projected) by using INCLUDE or ALL. You must define a RCU/WCU for the index. Note that, if the writes are throttled on the GSI, the main table will be throttled (Even if the WCU on the main table is fine) so be careful on what to choose as your GSI partition key and assign enough WCU to it.
  - You can add or modify GSI but not LSI.
- DynamoDB Accelerator (DAX)
  - Seamless cache for DynamoDB, no application rewrite (You just have to enable it).
  - Writes go through DAX to DynamoDB.
  - Micro second latency for cached reads and queries (it solves the hot key problem).
  - 5 minutes TTL for cache by default.
  - Up to 10 nodes (a virtual server) in the cluster.
  - Multi AZ (3 nodes minimum is recommended for production).
- Streams
  - Any changes in DynamoDB (create, update, delete) can end up in a DynamoDB stream (you can access to old item (image), new item (image), or the key attributes of the modified item).
  - This stream can be read by AWS Lambda, and we can react to changes in real time (send welcome email to new users), do some analytics, create derivative tables/views, implement cross region replication, insert into ElasticSearch.
  - This stream is a changelog and has 24 hours of data retention.
  - DynamoDB triggers connect DynamoDB streams to Lambda functions.
- TTL
  - Automatically deletes an item after an expiry date/time (it must be an epoch number).
  - TTL is provided at no extra cost, deletion do not use WCU/RCU.
  - TTL is a background task operated by the DynamoDB service itself.
  - Helps reduce storage and manage the table size over time.
  - TTL is enabled per row (You define a TTL column and add a date there).
  - DynamoDB typically deletes expired items within 48 hours of expiration.
  - Deleted items due to TTL are also deleted from GSI/LSI.
  - DynamoDB streams can help recover expired items.
- CLI

  - `--projection-expression`: attributes to retrieve.
  - `--filter-expression`: filter results.
  - `--page-size`: full dataset is still received but each API call will request less data (helps avoid timeouts).
  - `--max-items`: max number of results returned by the CLI, returns NextToken.
  - `--starting-token`: specifies the last received NextToken to keep on reading.

  ```dos
  aws dynamodb scan --table-name UserPosts --projection-expression "user_id, content" --region us-east-1
  ```

  ```dos
  aws dynamodb scan --table-name UserPosts --filter-expression "user_id = :u" --expression-attribute-values '{ ":u": {"S":"u123456"}}' --region us-east-1

  {
    "Items": [
      {
        "content": {
          "S": "Hello"
        },
        "user_id": {
          "S": "u123456"
        },
        "post_ts": {
          "S": "2018-09-02T12:34:00Z"
        }
      },
      {
        "content": {
          "S": "Hello Again"
        },
        "user_id": {
          "S": "u123456"
        },
        "post_ts": {
          "S": "2018-04-02T12:34:00Z"
        }
      }
    ],
    "Count": 2,
    "ScannedCount": 3,
    "ConsumedCapacity":null
  }
  ```

  - The following makes one API call:

  ```dos
  aws dynamodb scan --table-name UserPosts --region us-east-1
  ```

  - The following makes three API call (the result is the same as above):

  ```dos
  aws dynamodb scan --table-name UserPosts --region us-east-1 --page-size 1
  ```

  - The following makes one API call and receives just 1 item:

  ```dos
  aws dynamodb scan --table-name UserPosts --region us-east-1 --max-items 1
  ```

- Transactions
  - Ability to modify multiple rows in different table at the same time.
  - It is an "all or nothing" type of operation.
  - It consumers 2 x of WCU/RCU.
  - For example, account_balance and account_transactions table in the banking system.
- Other
  - VPC endpoints available to access DynamoDB without internet.
  - Access fully controlled by IAM.
  - Encryption at rest using KMS.
  - Encryption at flight using SSL/TLS.
  - Backup and restore feature is available to a point in time like RDS which has no performance impact (It is based on streams I believe).
  - Global tables which are multi-region, fully replicated, and high performance (It is based on streams I believe).
  - Amazon DMS can be used to migrate to DynamoDB (from MongoDB, MySQL, S3, Oracle, etc.).
  - You can launch a local DyanmoDB on your computer for development purposes.

## API Gateway

- Client interacts with Amazon API Gateway via REST API and API Gateway proxy the requests to Lambda.
- Lambda + API Gateway = No infrastructure to manage.
- Handles API versioning (v1, v2, ...).
- Handles different environments (dev, test, prod) and manages deployments.
- Handles security so we can remove security concerns out of Lambda and API Gateway can do it instead.
- Can create API keys and handles request throttling (if we are selling our API).
  - We first should create a usage plan
    - Throttling: set overall capacity and burst capacity
    - Quotas: number of requests made per day/week/month
    - Associate with desired API stages
  - API keys can be generated per customer and be associated with usage plans.
  - Ability to track usage for API keys.
- It has a very tight and neat integration with Swagger/OpenAPI and that allows us to import some files and quickly define our APIs and even export them as Swagger or OpenAPI3.
- Can help us transform and validate requests and responses.
- Generate SDK (for Android, JavaScript, Java, Ruby, iOS) and API specifications.
- You can cache API responses and limit the load on Lambda functions.
  - Default TTL is 300 secs (min: 0s, max: 3600S)
  - Caches are defined at stage level.
  - Caches encryption option is available.
  - Cache capacity is between 0.5GB to 237GB.
  - possible to override cache settings for specific methods at the resource level.
  - possible to flush the entire cache (invalidate it) immediately.
  - some clients (with proper IAM authorization) can invalidate the cache with header `Cache-Control:max-age=0`.
- Outside of a VPC, API Gateway can integrate with Lambda (most popular/powerful), endpoints on EC2, ELBs, external and publicly accessible HTTP endpoints, any AWS services, and inside of a VPC it can integrate with Lambda in your VPC and EC2 endpoints in the VPC.
- Create a new API > Give it a name and endpoint type of Regional > Choose Create method on the Actions > Choose GET > Choose Lambda to integrate API with (you have to create the Lambda function beforehand) and Save
- In the above example, you could define a resource (which is a path) and define methods in that path and integrate it with different Lambda functions.
- Stages
  - Making changes in the API Gateway does not mean they are effective.
  - You need to make a `deployment` for them to be in effect.
  - Changes are deployed to `Stages` (as many as you want but dev, test, prod is common).
  - Each stage has its own configuration parameters.
  - Stage variables are like environment variables for API Gateway.
  - Use stage variables to change often changing configuration values.
  - Stage variables can be used in Lambda functions ARN, HTTP endpoints (configure HTTP endpoints that your stages talk to -> We create a stage variable (such as alias) to indicate the corresponding Lambda alias. So API gateway will autmoatically invoke the right Lambda function -> dev stage talks to dev HTTP API.), and parameter mapping templates (Pass configuration parameters to Lambda through mapping templates. So Lambda knows if it's in dev test or prod. Note that stage variables are passed to the `context` object in Lambda as well.).
    ![](/md/pattern.jpg)
  - Stages can be rolled back as history of deployment is kept.
  - You can have some sort of blue/green deployment with Lambda & API gateway which is called `canary` deployment.
  - You choose the % of traffic the canary channel (stage) receives (new or green version of API).
  - Metrics and logs are separated for better monitorings.
  - You can overrid stage variables for canary stage.
  - Choose Deploy API from Actions > [New Stage] > Give it a name like dev and click Deploy > We will be given a URL that we can invoke `https://754snjsdf.execute-api.us-east-1.amazonaws.com/dev`.
- Mapping Templates
  - They can be used to modify request and/or response -> rename parameters, modify body content, add headers, map JSON to XML for sending to backend or back to client, and filter output results and remove unnecessary data.
  - It uses Velocity Template Language (VTL): for-loop, if, etc.
  - Click on the endpoint > Click on the Integration Response > Click on application/json > Choose Empty for Generate template > \$inputRoot is what is sent back by the Lambda function.
- Swagger/OpenAPI
  - Common way of defining REST APIs, using API definition as code.
  - Import existing Swagger/OpenAPI 3.0 spec to API Gateway
    - Method
    - Method Request
    - Integration Request
    - Method Response
    - (+) AWS extensions for API gateway and setup every single option
  - Can export current API as Swagger/OpenAPI spec.
  - Swagger can be written in YML or JSON.
  - Using Swagger we can generate SDK for our applications.
- Logging, Monitoring, and Tracing
  - CloudWatch Logs:
    - enable CW logging at the stage level (with log level).
    - override settings on a per API basis (ex: ERROR, DEBUG, INFO)
    - Log contains info about request and response body
  - CloudWatch Metrics:
    - Metrics are by stage
    - possibility to enable detailed metrics
  - X-Ray:
    - enable tracing to get extra info about requests made from API gateway to Lambda functions or DynamoDBs
    - X-Ray + API Gateway + Lambda gives you the full picture.
- CORS
  - CORS must be enabled when you receive API calls from another domain.
  - The OPTIONS (an HTTP method like GET) pre-flight request must contain the following headers:
    - Access-Control-Allow-Methods
    - Access-Control-Allow-Headers
    - Access-Control-Allow-Origin
  - CORS can be enabled through the console -> Actions > Enable CORS
- Security
  - IAM Permissions
    - Creates an IAM policy authorization and attach to client (AWS user or role).
    - Client uses `Sig v4` where IAM credentials are in headers.
    - API Gateway verifies access by contacting IAM and then goes to backend.
    - Good to provide access within your own infrastructure.
  - Lambda Authorizer (formerly known as Custom Authorizer)
    - Uses Lambda to validate the token (3rd party token) in header being passed.
    - You will pay per Lambda invocation. But there is an option to cache the result of authentication say for one hour.
    - Helps to use OAuth/SAML/3rd party type of authentication.
    - Lambda must return an IAM policy (very flexible in terms of what IAM policy is returned) for the user to the Gateway and it checks it out with the IAM and everything is good, it talks to the backend.
  - Cognito User Pools
    - Cognito fully manages user lifeclycle. No need to write any custom code.
    - You manage your own user pool (can be backed by facebook or Google login, etc.).
    - API gateway verifies identity automatically from AWS Cognito.
    - Cognito only helps with authentication and not authorization. Must implement authorization in the backend.
      ![](/md/cognito.jpg)
- Cognito

  - We want to give our users an identity so that they can interact with our app.
  - Cognito User Pools
    - Sign-in functionality for app users.
    - Integrates with API gateway.
    - Creates a serverless database of users.
    - Simple login: username or email and password combination.
    - Possibility to verify emails/phone numbers and add MFA.
    - Can enable Federated Identities (Facebook, Google, SAML, ...) -> You can allow users to login through these providers and get an identity into your user pool. Do not mistake this with Federated Identities (below).
    - Sends back a JWT.
    - Can be integrated with API Gateway for the authentication part.
  - Cognito Federated Identity Pools

    - Provides AWS credentials to users so that they can access AWS resources directly.
    - Integrates with Cognito User Pools an an identity provide.
    - Login to Federated Identity provider (or remain anonymous) -> Get temporary AWS credentials (from STS) back from Federated Identity Pool -> These credentials come with a pre-defined IAM policy stating their permissions.

    ![](/md/federated.jpg)

## Serverless Application Model (SAM)

- SAM is a serverless development framework which allows you to deploy your serverless app onto AWS.
- All the configuration will be a YML code.
- It generates complex CloudFormation from simple SAM YML file.
- Supports anything from CF: outputs, mappings, parameters, resources, ...
- Only two commands to deploy to AWS.
- SAM can use CodeDeploy to deploy Lambda functions if we want to be super picky of how we want to deploy our functions.
- SAM can help you run Lambda, API Gateway, and DynamoDB locally.
- Recipe for SAM template
  - Transform header indicates it's a SAM template: `Transform: 'AWS::Serverless-2016-10-31'`
  - Write Code, for each resources we can have the following `Type`:
    - `Type: 'AWS::Serverless::Function'` -> Lambda function
    - `Type: 'AWS::Serverless::Api'` -> API Gateway
    - `Type: 'AWS::Serverless::SimpleTable'` -> DynamoDB
  - Package & Deploy
    - `aws cloudformation package` or `sam package`
    - `aws cloudformation deploy` or `sam deploy`
- So we have our SAM template YML file + Application code + Swagger file (optional, required to have an API Gateway) ->

  - `aws cloudformation package` will transform SAM template to CF template and upload the zip file of the code to S3 bucket. This generated template will have a reference to the uploaded code in S3.
  - When we run `aws cloudformation deploy` will do a deployment of the generated template. It creates and executes a change set. A change set is basically figuring out how CF should take its existing state and move it to to next state based on the modifications we have generated. CF will apply the change set to our stack (API Gateway, Lambda, DynamoDB, IAM policies).

![](/md/sam.jpg)

- Install sam CLI.
- Create a `gen` folder and a `src` folder with one file `app.py` in it for the Lambda function.

```py
import json
import boto3
import os

print('Loading function')
# create the client outside of the handler
region_name = os.environ['REGION_NAME']
dynamo = boto3.client('dynamodb', region_name=region_name)
table_name = os.environ['TABLE_NAME']

def respond(err, res=None):
  return {
    'statusCode': '400' if err else '200',
    'body': err.message if err else json.dumps(res),
    'headers': {
      'Content-Type':'application/json',
    },
  }

def lambda_handler(event, context);
  return respond(None, res=dynamo.scan(TableName=table_name))
```

- Create a `template.yml` file in the root:

```yml
AWSTemplateFormatVersion: "2010-09-09"
Transform: "AWS::Serverless-2016-10-31"
Description: A starter AWS Lambda function.
Resources:
  helloworldpython3:
    Type: "AWS::Serverless::Function"
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.6
      CodeUri: src/
      Description: A starter AWS Lambda function.
      MemorySize: 128
      Timeout: 3
      Environment:
        Variables:
          TABLE_NAME: !Ref MyTable
          REGION_NAME: !Ref AWS::Region
      Events:
        HelloWorldSAMAPI:
          Type: Api
          Properties:
            Path: /hello
            Method: GET
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref MyTable
  MyTable:
    Type: "AWS::Serverless::SimpleTable"
    Properties:
      PrimaryKey:
        Name: greeting
        Type: String
      ProvisionedThroughput:
        ReadCapacityUnits: 1
        WriteCapacityUnits: 1
```

- Run the following commands in the terminal:

```dos
aws s3 mb s3://john-code-sam
```

```dos
aws cloudformation package --s3-bucket john-code-sam --template-file template.yml --output-template-file gen/template-generated.yml
```

```dos
aws cloudformation deploy --template-file gen/template-generated.yml --stack-name hello-world-sam-stack --capabilities CAPABILITY_IAM
```

- There are other policy templates to apply permissions to your Lambda functions:
  - S3ReadPolicy: gives read only permissions to objects in S3.
  - SQSPollerPolicy: allows to poll an SQS queue
  ```YML
  Policies:
        - SQSPollerPolicy:
            QueueName: !GetAtt MyQueue.QueueName
  ```
  - DynamoDBCrudPolicy: you've seen above.

# Elastic Container Service, ECS (Docker in AWS)

- ECR (Elastic Container Registery) is a private repository to store the Docker images of your app.
  ![](/md/docker.jpg)
- Docker containers management:
  - ECS: Amazon's own platform
  - Fargate: Amazon's own serverless platform
  - EKS: Amazon's managed Kubernetes (open source) -> In IC Markets, we use this.

## ECS Clusters

- ECS clusters are logical grouping of EC2 instances (container instances) that run a special AMI which have Docker installed.
- EC2 instances run the ECS agent in a docker container.
- The EC2 instance has a user data that writes the cluster it belongs to on startup to `/etc/ecs/ecs.config` file.
- The ECS agent looks into that file and register the EC2 instance to the ECS cluster.
- ECS cluster is similar to Elastic Beanstalk (I wrote this!).

## ECS Task Definition

- Tasks definitions are metadata in JSON format to tell ECS how to run a docker container (for example an apache server inside an EC2).
- Tasks can have IAM role for example to pull image from ECR or talk to S3.
- It has information around
  - Image name
  - Port binding for container (e.g. port 80) and host (EC2) (e.g. port 8080)
  - Memory and CPU required (it should be less than the memory and cpu of the host EC2 instance obviously)
  - Environment variables
  - Network information

## ECS Service

- You can create ECS services at the level of cluster.
- Each service has one task definition.
- Service type can be replica (places and maintains the desired number of tasks across your cluster) or daemon (deploys exactly one task on each active container instance).
- ECS Service helps define how many tasks should run and how they should be run.
- They ensure that the number of tasks desired is running across our fleet of EC2 instances.
- They can be linked to ELB/NLB/ALB if needed.
- Don't forget to allow 8080 in the SG of the container instance in case of apache.
- In case of apache, we cannot have more than one task in the service to be run on one container instance (because it uses port 8080 and for the second task the port is already in use) but we can scale ECS instances and have two container instances in our cluster to run apache server each.
- If we want to have many tasks running on many instances (more than one in each instance), we should change the task definition and not include any host port so it should be assigned randomly.

  ![](/md/ecs.jpg)

- To do this, go to `task definition`, select the previous task and click on `create new revision` and remove the host port. Then, update the service to use the newer version.
- We cannot update the service and add a LB now. So we have to create another service and add an Application Load Balancer. For this, we have to create an ALB first (give it a dummy target group). Then, we have to edit the SG of EC2 containers and allow any traffic from the ALB that we just created. When we selected the new ALB, we have to select the container to load balance (in it, select `create new` for the target group name and `/` for path pattern and health check).

## ECS Summary

- EC2 instanes must be created.
- We must configure the file `etc/ecs/ecs.config` with the cluster name.
- The EC2 instance must run an ECS agent.
- EC2 instances can run multiple containers of the same type:
  - You must not specify a host port (only container port).
  - You should use an application load balancer with the dynamic port mapping.
  - The EC2 instance SG must allow traffic from the ALB on all ports.
- ECS tasks can have IAM roles to execute actions against AWS.
- SGs operate at the instance level, not task level.

## ECR

- So far we've been using Docker images from Docker hub.
- ECR is a private docker image repository.
- Access is controlled through IAM policies.
- You need to run some commands to push/pull:
  - Create a docker image using docker build command and a Dockerfile.
  ```dos
  docker build -t demo .
  ```
  - Create a repository in ECR like demo.
  - For Mac and Linux:
  ```dos
  $(aws ecr get-login --no-include-email --region eu-east-1)
  ```
  - The command in the () above generates this command `docker login -u AWS -p mnbvmxvbxmvbskfh... https://1234567890.dkr.ecr.eu-west-1.amazonaws.com` which is a temporary docker login that we can use to login to our ECR repository.
  - Tag your image so you can push the image to the above repository.
  ```dos
  docker tag demo:latest 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
  ```
  ```dos
  docker push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
  ```
  ```dos
  docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
  ```
  - Now you can create a new revision of the task definition to use our own image stored in ECR in the container definition section. Then update the service to use the new revision.

## Fargate

- When launching an ECS cluster, we had to create our EC2 instance and when scaling, we had to add EC2 instances so we were managing infrastructure.
- With Fargate, it's all serverless and we don't provision EC2 instances. So create a Fargate cluster.
- We just create a task definition (compatible to Fargate) and create a service out of it (Choose the SG that allows traffic from ALB and also choose the ALB and add this container to it. Don't forget to remove the existing path pattern from the listener if you are using the same ALB as before.) and AWS will run our containers for us.
- To scale, just increase the task number.
- Fargate containers are provisioned by the container spec (CPU/RAM).
- Here, Fargate tasks can have IAM roles to execute actions against AWS (unlike ECS tasks).

## EB + ECS

- You can run Elastic Beanstalk in single or mullti docker container mode.
- Multi docker helps run multiple containers per EC2 instance in EB.
- This will create for you:
  - ECS Cluster
  - EC2 instances, configured to use the ECS cluster
  - Load balancer (in high availability mode)
  - Task definitions and execution
- Requires a config file `Dockerrun.aws.json` at the the root of source code.
- Your docker images must be pre-built and stored in ECR for example.
  ![](/md/eb.jpg)
