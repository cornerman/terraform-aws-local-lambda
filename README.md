# terraform-aws-local-lambda

Imagine: You have a lambda somewhere inside AWS which reacts to EventBridge/SNS/SQS/ApiGateway/whatever.
Now you want to develop, run and debug this lambda locally without long deployment times in between.
Meet: local lambda development!

This module deploys a stub lambda instead of the actual one. This stub running inside of AWS will send each lambda request and its envrionment variables to a websocket connection.
Locally, you can connect to this websocket and get new requests, run them locally, and then send a response back into the websocket.
This response is then returned by the AWS lambda stub that is waiting for the response.

This repository contains two reusable modules:
- Deploy the environment needed to do local lambda debugging
- A lambda stub for each function you want to debug

This is heavily inspired by: [serverless-stack/sst: live lambda development](https://github.com/serverless-stack/sst/blob/master/www/docs/live-lambda-development.md). Though, we are using terraform instead of cdk/cloudformation. We also try to keep this as generic as possible, so that you can use it independent of your development stack. Additionally, we secure all debugging interfaces (API and S3 Bucket) with IAM authorization.

## Requirements

- terraform (>= 1.0.0)

## How to use?

### CLI Tool

Easiest way to run this is with our CLI tool. It sets up the local lambda environment and stub existing lambda function.

See: [local-lambda](https://github.com/fun-stack/local-lambda)

### Terraform

First create the needed environment for local lambda development. This is basically just a Websocket API Gateway to send requests from the AWS lambda to your computer and responses from your computer to the lambda. It is needed once for you app. It is completely serverless and usage will most probably be in the AWS free-tier or very cheap - so no worries to keep this running.

```terraform
module "local_lambda_environment" {
  source  = "fun-stack/local-lambda/aws//environment"
  version = "0.1.0"

  # prefix = "my-local-lambda-env"
}
```

You can then create a lambda stub instead of your normal lambda:
```terraform
module "my_local_lambda" {
  source  = "fun-stack/local-lambda/aws"
  version = "0.1.0"

  name        = "my-lambda-1"
  environment = module.local_lambda_environment

  role_arn = "..."
}
```

Outputs:
- `module.my_local_lambda.function_arn`
- `module.my_local_lambda.role_arn`
- `module.my_local_lambda.websocket_url`
- `module.my_local_lambda.websocket_secret`
- `module.local_lambda_environment.websocket_url`
- `module.my_local_lambda.role_arn`
- `module.my_local_lambda.websocket_url`
- `module.my_local_lambda.websocket_secret`

