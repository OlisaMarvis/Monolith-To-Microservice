# Monolith-To-Microservice

### Break a Monolith Application into Microservices with AWS Elastic Container Service, Docker, and Amazon EC2

In this project, I will deploy and walk you through how to deploy a monolithic node.js application to a Docker container, then decouple the application into microservices without any downtime. The node.js application hosts a simple message board with threads and messages between users.

### Why this process is important
As an application's code base grows and new features are added. It becomes exceedingly complex to update and maintain. Introducing new features, languages, frameworks, and technologies becomes very hard, limiting innovation and new ideas.

This issue is solved by breaking the traditional monlithic application into a microservices architecture. Here, each application component runs as its own service and communicates with other services via a well-defined API. Microservices are built around business capabilities, and each service performs a single function. Microservices can be written using different frameworks and programming languages, and you can deploy them independently, as a single service, or as a group of services.

### Application Architecture
As we progress, I will show you how to run a simple monolithic application in a Docker container, deploy the same application as microservices, then switch traffic to the microservices without any downtime.

![](images/App-Architecture)

**Monolithic Architecture**

As the picture above indicates, The entire node.js application is run in a container as a single service and each container has the same features as all other containers. If one application feature experiences a spike in demand, the entire architecture must be scaled.

**Microservices Architecture**

In this design, each feature of the node.js application in the microservice runs as a seperate service within it's own container. The services can scale and be updated independently of the others.

Now that you have the high level overview of the project. Let's get to it!

##Pre-requisites for this project
1. Have an Aws account.
2. Install Docker.
3. Install the AWS CLI
4. Have a text editor

**1. Have an AWS account.**
Create and register an AWS account using [this instruction](https://repost.aws/knowledge-center/create-and-activate-aws-account/)

**2. Install Docker.**
Install docker depending on your os. Here I'm using a MacOs. [Read instructions here](https://docs.docker.com/desktop/install/mac-install/)

**3. Install the AWS CLI**
Install AWS CLI on your system. [Read instructions here](https://docs.aws.amazon.com/cli/v1/userguide/install-macos.html)

**4. Have a code Editor**
Install Visual studio code. [Read instructions here](https://code.visualstudio.com/docs/setup/mac)

This project will consist of 5 modules.

**Module 1:** Containerise the Monolith
**Module 2:** Deploy the Monolith
**Module 3:** Break the Monolith
**Module 4:** Deploy the Monolith
**Module 5:** Clean up

We'll be taking each module step by step from beginning till the end of the project.
### Module 1 - Containerise the Monolith

Here, I will build the container image for your monolithic node.js application and push it to Amazon Elastic Container Registry.
The picture below demonstatrates how the whole process will go.

![](images/module-diagram.png)

Before we go ahead with this module it's important we understand some concepts.

**Container** 
What is container? 
A container is a unit of software that packages code and its dependencies so the application runs quickly and reliably across computing environments.

One may ask, why use Containers?

Most companies use containers for their application because of:
**Everything in a single package:** A Docker image contains all of an application’s binaries, dependencies and libraries – everything the application needs to run. There’s no need to run an install process.

**Repeatability:** A container image will run the same way, wherever it is run. So once my application is built as a Docker image, I can run my container wherever Docker is installed. I can also run multiple instances of my application very easily, and they will all behave in the same way.

**Lightweight:** Containers are cheap, and fast to create and destroy. This makes it easier to upgrade and patch software in containers. You don’t modify or upgrade an app inside a container when it’s running: you just kill the old container, and launch a new one with the upgraded application.

**Portability:** With Docker, portability comes from the Docker image format. It’s like a zip file (actually it’s a TAR archive) which contains a whole application and all of its dependencies. You can download Docker images from public or private registries, or create your own. A container will run the same on a laptop, in the data center, or in the public cloud.

**Isolation:** A Docker container offers a certain level of isolation. This means that an application runs in its own sealed environment, and cannot affect other applications when it’s running inside a container, unless you choose to allow it.


Now let's get to the job.
After completing the instructions in the prerequisities. Navigate to https://github.com/awslabs/amazon-ecs-nodejs-microservices. Open your terminal or visual studio and clone this repo https://github.com/awslabs/amazon-ecs-nodejs-microservices.git, to our local environment. 

In your project folder, you should see folders for infrastructure and services. Infrastructure holds the AWS CloudFormation infrastructure configuration code you will use in the next step. Services contains the code that forms the node.js application.

Take a few minutes to review the files and familiarize yourself with the different aspects of the application, including the database db.json, the server server.js, package.json, and the application Dockerfile.

Next, let's navigate to our AWS and create an IAM user who'll have access to create the resources we need. The steps are listed in the pictures shown below.

When we are done, we'll have to login with the IAM user credentials which we downladed from the last step. 

Then

- Navigate to the [Amazon ECR console](https://console.aws.amazon.com/ecs/home?#/repositories).
- On the *Repositories* page, select *Create Repository*.
- On the Create repository page, enter the following name your repository: api *Note:* Under *Tag immutability*, leave the default settings.
- Select *Create repository*.

The steps are outlined with the attached pictures attached below.

![](images/module-diagram.png)
![](images/module-diagram.png)

- Build and push docker image: Use the terminal to authenticate Docker log in:

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 283451138393.dkr.ecr.us-east-1.amazonaws.com
```
![](images/module-diagram.png)

if needed, configure your AWS credentials.
You can see the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

If after executing the aws configure command, still getting an permission denied, type sudo in front of the docker login command.

```
sudo aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 283451138393.dkr.ecr.us-east-1.amazonaws.com
```
![](images/module-diagram.png)

After this, make sure your inside the api folder before running the next command. Access your terminal and navigate to the following directory: ~/amazon-ecs-nodejs-microservices/2-containerized/services/api.

1. To build the image, run the following command in the terminal: 
``` 
docker build -t api . 
```
**Note:** The period (.) after api is needed.

2. After the build completes, tag the image so you can push it to the repository: docker tag 
```
api:latest [account-ID].dkr.ecr.[region].amazonaws.com/api:v1 
```
**Note:** Replace the [account-ID] and [region] placeholders with your specific information.
**Pro tip:** The :v1 represents the image build version. Every time you build the image, you should increment this version number. If you were using a script, you could use an automated number, such as a time stamp to tag the image. This is a best practice that allows you to easily revert to a previous container image build in the future.

3. Push the image to Amazon ECR by running: docker push [account-id].dkr.ecr.[region].amazonaws.com/api:latest
**Note:** replace the [account-ID] and [region] placeholders with your specific information.
If you navigate to your Amazon ECR repository, you should see your image tagged latest.

![](images/module-diagram.png)
![](images/module-diagram.png)

### Module 2 - Deploy the monolith

In this module, you will use Amazon Elastic Container Service (Amazon ECS) to instantiate a managed cluster of EC2 compute instances and deploy your image as a container running on the cluster.

### Architecture Overview

1. Client: The client makes a request over port 80 to the load balancer.

2. Load Balancer: The load balancer distributes requests across all available ports.

3. Target Groups: Instances are registered in the application's target group.

4. Container Ports: Each container runs a single application process which binds the node.js cluster parent to port 80 within its namespace.

5. Containerized node.js Monolith: The node.js cluster parent is responsible for distributing traffic to the workers within the monolithic application. This architecture is containerized, but still monolithic because each container has all the same features of the rest of the containers.

### Amazon Elastic Container Service: Definition
Amazon ECS is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. It deeply integrates with the rest of the AWS platform to provide a secure and easy-to-use solution for running container workloads in the cloud and now on your infrastructure with Amazon ECS Anywhere.

There is no additional charge for Amazon ECS. You pay for the AWS resources (for example, EC2 instances or EBS volumes) you create to store and run your application.

*Services Used:*

- [Amazon Elastic Container Service](https://aws.amazon.com/ecs/)
- [Amazon Elastic Container Registry](https://aws.amazon.com/ecr/)
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
- [Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/)

### Implementation Instructions

Follow the step-by-step instructions below to deploy the node.js application using Amazon ECS.

Create an Amazon ECS cluster deployed behind an Application Load Balancer. In order to do so, let's navigate to the AWS CloudFormation console. [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

**Step 1:** Create a Stack



1. Select Create stack.
2. Select Upload a template file and choose the ecs.yml file from the GitHub project. 
   At amazon-ecs-nodejs-microservice/   2-containerized/infrastructure/ecs.yml then select Next.

3. For the stack name, enter BreakTheMonolith-Demo. Verify that the other parameters have the following values:

    Desired Capacity = 2
    InstanceType = t2.micro
    MaxSize = 2
    Select Next.

3. On the Configure stack options page, keep the default options and scroll down and select Next.

4. On the Review BreakTheMonolith-Demo page scroll to the bottom of the page. 
   Acknowledge the Capabilities statement by selecting the checkbox, and select Create stack.

5. You will see your stack with the status CREATE_IN_PROGRESS. 
    You can select the refresh button at the top right of the screen to check on the progress. This process typically takes under 5 minutes.

    ![](images/module-diagram.png)

**NOTE:** Optionally, you can use the AWS Command Line Interface (AWS CLI) to deploy AWS CloudFormation stacks. Run the following code in the terminal from the folder amazon-ecs-nodejs-microservices/3-microservices and replace [region] with your AWS Region.

```
$ aws cloudformation deploy \
   --template-file infrastructure/ecs.yml \
   --region [region] \
   --stack-name BreakTheMonolith-Demo \
   --capabilities CAPABILITY_NAMED_IAM
```

**Step 2.** Check the Cluster is Running

Let's navigate to the [Amazon ECS console](https://console.aws.amazon.com/ecs/home?). My cluster should appear in the list.

![](images/module-diagram.png)

Select the cluster BreakTheMonolith-Demo, then select the Tasks tab to verify that there are no tasks running.

![](images/module-diagram.png)

- Select the ECS Instances tab to verify there are two Amazon EC2 instances created by the AWS CloudFormation template.

**Note:** If you receive a message that the ECS agent is outdated, select Learn more for instructions to update the ECS agent.

![](images/module-diagram.png)

**Step 3.** Specify the task definition

Task definitions specify how Amazon ECS deploys the application containers across the cluster.

By default, this should be created already if you followed all the steps. But if it's not there, you can follow the steps below to create a task.

- From the Amazon ECS left navigation menu, select Task Definitions.
- Select Create new Task Definition.
- On the Select launch type compatibility page, select the EC2 option then select Next step.
- On the Configure task and container definitions page, do the following:
- In the Task Definition Name field, enter api.
- Scroll down to Container Definitions and select Add container.
- In the Add container window:
- Parameters that are not defined can be either left blank or with the default settings.
- In the Container name field, enter api.
- In the Image field, enter [account-ID].dkr.ecr.[region].amazonaws.com/api:v1
- Replace [account-ID] and [region] with your specific information. Ensure the tag v1 matches the value you used in Module 1 to - tag and push the image. This is the URL of your ECR repository image that was created in the previous module.
- In the Memory Limits field, verify Hard limit is selected and enter 256 as the value.
- Under Port mappings, Host port = 0 and Container port = 3000.
- Scroll to ENVIRONMENT, CPU units = 256.
- Select Add.
- You will return to the Configure task and container definitions page.
- Scroll to the bottom of the page and select Create.
- Your Task Definition is listed in the console.

By the time you are done, you should have some thing like this.

![](images/module-diagram.png)


**Step 4.** Configure the load Balancer target group

**Check your VPC Name:**

In order to do so, let's
- Navigate to the [Load Balancer section of the EC2 Console](https://console.aws.amazon.com/ec2/v2/home?#LoadBalancers:).

The Application Load Balancer (ALB) lets your service accept incoming traffic. The ALB automatically routes traffic to container instances running on your cluster using them as a target group.

Check your VPC Name: If this is not your first time using this AWS account, you may have multiple VPCs. It is important to configure your Target Group with the correct VPC.

- Navigate to the Load Balancer section of the EC2 Console.
- Locate the Load Balancer named demo/starts with Break.
- Select the checkbox next to it to see the Load Balancer details.
- In the Description tab, locate the VPC attribute (in this format: vpc-xxxxxxxxxxxxxxxxx).
- copy this vpc id to your notepad, we'll be needing it later.

![](images/module-diagram.png)


**Note:** You will need the VPC attribute in the next step when you configure the ALB target group.

Configure the ALB Target Group

- Navigate to the Target Group section of the EC2 Console.
- Select Create target group.
- Configure the following Target Group parameters (for the parameters not listed below, keep the default values):
- For the Target group name, enter api.
- For the Protocol, select HTTP.
- For the Port, enter 80.
- For the VPC, select the value that matches the one from the Load Balancer description. This is most likely NOT your default VPC.
- Access the Advanced health check settings and edit the following parameters as needed: 
- For Healthy threshold, enter 2.
- For Unhealthy threshold, enter 2.
- For Timeout, enter 5.
- For Interval, enter 10.
- For sucess codes, enter 200.
- Select Create.

![](images/module-diagram.png)

**Step 5.** Configure the load Application Load Balancer: Listener

The ALB listener checks for incoming connection requests to your ALB.

Add a Listener to the ALB

- Navigate to the Load Balancer section of the EC2 Console.
- Select the checkbox next to demo to see the Load Balancer details.
- Select the Listeners tab.
- Select Add listener and edit the following parameters as needed:
- For Protocol:port, select HTTP and enter 80.
- For Default action(s), select Forward to and in the Target group field, enter api.
- Select Save.

![](images/module-diagram.png)

**Step 6.** Deploy the monolith as a service into the cluster

Let's navigate to the [Amazon ECS console](https://console.aws.amazon.com/ecs/home?).

- Select the cluster BreakTheMonolith-Demo, select the Services tab then select Create.
- On the Configure service page, edit the following parameters (and keep the default values for parameters not listed below): 
- For the Launch type, select EC2.
- For the Service name, enter api. 
- For the Number of tasks, enter 1.
- Select Next step.
- On the Configure network page, Load balancing section, select Application Load Balancer.
- Additional parameters will apear: Service IAM role and Load balancer name.
- For the Service IAM role, select BreakTheMonolith-Demo-ECSServiceRole.
- For the Load balancer name, verify that demo is selected.
- In the Container to load balance section, select Add to load balancer.
- Additional information labeled api:3000 is shown.
- In the api:3000 section, do the following:
- For the Production listener port field, select 80:HTTP.
- For the Target group name, select your group: api.
- Select Next step.
- On the Set Auto Scaling page, leave the default setting and select Next step.
- On the Review page, review the settings then select Create Service.
- After the service has been created, select View Service.