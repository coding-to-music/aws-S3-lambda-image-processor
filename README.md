# Image Processor Function

https://github.com/coding-to-music/aws-S3-lambda-image-processor

Written by Jason Conway-Williams

JCDubs

https://github.com/JCDubs/Image-Processor

https://jcdubs.medium.com/s3-image-processing-lambda-b75052b1dd5e

The Image Processor Function provides functionality to resize and convert images through AWS S3 and lambda.

## Setup

The project is provided as a Node.js AWS lambda project, written withe a base version of Node.js 12 >. You should perform the following steps to set the project up.

- Clone the project locally.
- Install NVM if it is not already installed and set the node version as 12.
- Run `yarn` to install all dependencies.

Before deployment, a sandbox, static site and deployment S3 Buckets should be created and the bucket names should be stated in th serverless.yml file.

## Environments

The `serverless.yml` file has been configured to deploy the lambda to a dev, test or prod environment which is achieved by running the sls deploy command specifying the stage to deploy to. By default, if not specified, the stage is set to 'dev'.

Variables used in serverless.yml:

```java
bucket: ${self:custom.s3.imageSandboxBucketName.${opt:stage, 'test'}}

stage: ${opt:stage, 'test'}
region: ${opt:region, 'eu-west-1'}

name: ${self:custom.deploymentBucket.${self:provider.stage}}

IMAGE_SANDBOX_BUCKET: ${self:custom.s3.imageSandboxBucketName.${self:provider.stage}}
SITE_IMAGES_BUCKET: ${self:custom.s3.imageSiteBucketName.${self:provider.stage}}
ENV: ${self:custom.env.${self:provider.stage}}
VERSION: ${file(./package.json):version}
Resource: "arn:aws:s3:::${self:custom.deploymentBucket.${self:provider.stage}}/\*"
          "arn:aws:s3:::${self:custom.s3.imageSiteBucketName.${self:provider.stage}}/*"
Resource: "arn:aws:s3:::${self:custom.s3.imageSandboxBucketName.${self:provider.stage}}/*"
    environment: ${self:provider.stage}
    application: ${self:service}
    product: ImageProcessor
```

## Deployment

It is important that the Sharp node module is installed on the operating system it is intended to be run on. A Dockerfile has been provided within this project to deploy the lambda to AWS. The Dockerfile is configured to run in a similar OS to AWS Lambda ensuring that the Sharp node module is installed in the Docker container OS. The following commands should be run to build the docker image and deploy.

`docker build -t image-proc .`

The lambda should then be deployed with the following command providing the AWS credentials in a volume unless the machine the docker container is being run on has been configured with an IAM role.

`docker run -v ~/.aws:/root/.aws image-proc sls deploy -v -s test`

```java
Running "serverless" from node_modules
Serverless: Packaging service...
Serverless: Excluding development dependencies...

  Serverless Error ---------------------------------------

  Could not locate deployment bucket. Error: Access Denied

  Get Support --------------------------------------------
     Docs:          docs.serverless.com
     Bugs:          github.com/serverless/serverless/issues
     Issues:        forum.serverless.com

  Your Environment Information ---------------------------
     Operating System:          linux
     Node Version:              12.20.1
     Framework Version:         1.63.0
     Plugin Version:            3.3.0
     SDK Version:               2.3.0
     Components Core Version:   1.1.2
     Components CLI Version:    1.4.0
```

## Running Locally

The Lambda can be run locally by executing `yarn serve`. The fist time the lambda is run locally, a `buckets` directory will be created in the root of the project containing the test and development equivalent buckets.

## Test

### Unit Tests

Tests are located in the `test/unit` directory consisting of unit tests. The tests can be run by executung the `yarn test` command.

### Integration Tests

The integration tests are located in the `test/integration` directory consisting of cucumber tests. The tests use the local s3 buckets to execute and validate the process of uploading an image to S3, generating the jpeg variant of the image in the local static site bucket. Images can be deleted from the local buckets using the `yarn delete-canned` command. Images can be uploaded to the local buckets using the `yarn upload-canned` command.
