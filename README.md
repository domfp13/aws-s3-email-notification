# AWS-S3-EMAIL-NOTIFIER

Based on a S3 bucket event (PUT, POST), sends an email notification through AWS SNS. 

This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders.

- EventNotification - Code for the application's Lambda function and Project Dockerfile.
- events - Invocation events that you can use to invoke the function.
- EventNotification/tests - Unit tests for the application code.
- template.yaml - A template that defines the application's AWS resources.

The application uses few AWS resources, including Lambda functions, SNS, S3, CloudWatch Logs. These resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates this application.

![Application Diagram](https://user-images.githubusercontent.com/7782876/151227559-81971f61-0fe1-4e4b-a6af-576014ada0a7.png)

## Run application locally

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

To use the SAM CLI, you need the following tools.

* AWS CLI - [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
* SAM CLI - [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community)
* AWS Account with Root rights to create services.

This repo contains a `Makefile` to build and test (locally) and deploy your application, run the following in your shell:

```bash
aws-s3-email-notifier$ make help
```

This project is deployed using CloudFormation, it requires a few variables in order to create the resources, the first thing you will need is an Elastic Cloud Repository, this will be used to host the container image. 
To create an ECR (Elastic Container Registry) run the following command, and copy the URI (IMAGE-REPOSITORY) that its generated in the `.env` file of this project.
```bash
aws ecr create-repository \
    --repository-name s3-email-notifier \
    --image-tag-mutability IMMUTABLE \
    --image-scanning-configuration scanOnPush=true
```
The ECR gives you a place to upload the application image so that [AWS CloudFormation](https://aws.amazon.com/cloudformation/) can access the container image when it runs the deploy process.
If you want to get information about all available ECR you can run the following command:
```bash
aws ecr describe-repositories
```

CloudFormation will need 4 variables, you will need to fill out these variables under `.env` file within this repository, variables are the following:
* STACK-NAME=<YOUR_STACK_NAME>
* REGION=<AWS_REGION>
* IMAGE-REPOSITORY=<YOUR_ECR_URI>
* BUCKETNAME=<YOUR_BUCKET_NAME>
* TOPICNAME=<YOUR_SNS_TOPIC_NAME>
* ENDPOINTEMAIL=<YOUR_EMAIL_ADDRESS>

Once you fill out these variables, the next step is to build and run your application. The following command will compile and build `packaged-template.yaml` based of `template.yaml` file in `.aws-sam/build/` directory.
```bash
aws-s3-email-notifier$ make run
```

## Deploy Application to AWS
The SAM CLI reads the application template to determine the API's routes and the functions that they invoke. The `Events` property on each function's definition includes the route and method for each path.

```yaml
  Events:
    HelloWorld:
      Type: Api
      Properties:
        Path: /hello
        Method: get
```

To package and deploy the application run:

```bash
aws-s3-email-notifier$ make package
```
Finally to deploy the package run:
```bash
aws-s3-email-notifier$ make deploy
```

The command will build a docker image from a Dockerfile and then the source of your application inside the Docker image.
You can find information about the services created in the output values displayed after deployment.

## Add a resource to your application
The application template uses AWS Serverless Application Model (AWS SAM) to define application resources. AWS SAM is an extension of AWS CloudFormation with a simpler syntax for configuring common serverless application resources such as functions, triggers, and APIs. For resources not included in [the SAM specification](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md), you can use standard [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) resource types.

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
aws-s3-email-notifier$ sam logs -n <YOUR_FUNCTION_NAME> --stack-name <YOUR_STACK_NAME> --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Unit tests

Tests are defined in the `EventNotification/tests` folder in this project. Use NPM to install the [Mocha test framework](https://mochajs.org/) and run unit tests from your local machine.

```bash
aws-s3-email-notifier$ cd EventNotification
EventNotification$ npm install
EventNotification$ npm run test
```

## Cleanup

To delete the application, use the AWS CLI. Assuming you set up your project name for the stack name in the `.env` file, you can run the following:

```bash
aws-s3-email-notifier$ make undeploy
```

## Authors
* **Luis Enrique Fuentes Plata** - *2022-09-05*
