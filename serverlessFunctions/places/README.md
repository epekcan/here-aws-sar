# AWS SAM for HERE Location Service APIs - Places
## Introduction
This project provides [AWS Lambda](https://aws.amazon.com/lambda/) as __proxy__ for [HERE Places API](https://developer.here.com/documentation/places/topics/introduction.html). This AWS Lambda is packaged as per the [AWS Serverless Application Model](https://aws.amazon.com/about-aws/whats-new/2016/11/introducing-the-aws-serverless-application-model/).

"AWS SAM is natively supported by AWS CloudFormation and defines simplified syntax for expressing serverless resources. The specification currently covers APIs, Lambda functions and Amazon DynamoDB tables."

### Benefits

The AWS API Gateway supports configuring both Cache and Throttling, and the lambdas are open source: we welcome pull requests with circuit breakers, graceful error handling, etc.!

## Requirements
To successfully call the [HERE Places API](https://developer.here.com/documentation/places/topics/introduction.html) through the proxy in this project, you need to obtain HERE API credentials. Multiple plans are available: https://aws.amazon.com/marketplace/pp/B07JPLG9SR/?ref=_ptnr_aws_sar_github#pricing-information.

## Setup
### Step 1: Register for an API Key

Visit the [HERE Location Services on AWS Marketplace](https://aws.amazon.com/marketplace/pp/B07JPLG9SR/?ref=_ptnr_aws_sar_github), and review the [Access Control FAQ](https://developer.here.com/faqs#access-control).

### Step 2: Register an AWS Account

Visit [AWS](https://aws.amazon.com/free/) and sign up for a Free Tier account.

### Step 3: Install the AWS CLI and run "aws configure"

Download and install the [AWS CLI](https://aws.amazon.com/cli/), and run `aws configure` as per the [AWS CLI User Guide](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

### Step 4: Get the Source

From GitHub: clone [this repository](https://github.com/heremaps/here-aws-sar), or download the ZIP.

The folder containing the lambda source code (JS) and CloudFormation templates (YML) is `serverlessFunctions\places\`.

### Step 5: Package

An S3 bucket is required as a destination for the AWS SAM package. If you don't have one already, create one:

`aws s3 mb s3://here-maps-api--aws-sam`

Note: If the folder contains a `package.json` file: run `npm update`:

`x:\src\here-aws-repository\serverlessFunctions\places>npm update`

Use the AWS CLI to package (note the folder layout):

`x:\src\here-aws-repository\serverlessFunctions>aws cloudformation package --s3-bucket here-maps-api--aws-sam --template-file places\places.yml --output-template-file places-packaged.yml`

### Step 6: Deploy

Use the AWS CLI to deploy the AWS SAM package using CloudFormation:

`x:\src\here-aws-repository\serverlessFunctions>aws cloudformation deploy --capabilities CAPABILITY_IAM --stack-name "HERE-Maps-API--Places" --parameter-overrides HereApiKey=<apiKey> --template-file places-packaged.yml`

### Step 7: Find new API Gateway URL

Once deployment completes, look for the URL of the new API Gateway. It should follow this pattern:

`https://<apigw>.execute-api.<region>.amazonaws.com/...`

The API Gateway URL is an output from the CloudFormation template and can be found among the tabs when selecting a Stack in the AWS Console.

Alternatively look at the API Gateway in the AWS Console, select Stages, and then expand the tree until you see "Invoke URL".

### Step 8: Secure your API Gateways/Lambdas

Note: The AWS Lambda proxy deployed above do not impose **authentication** or **authorization** restrictions!

__You must decide how you will control access to your API Gateway and Lambda.__

For guidance, see the [AWS Lambda FAQ](https://aws.amazon.com/lambda/faqs/#security).

Consider implementing [AWS API Gateway Custom Authorizers](http://docs.aws.amazon.com/apigateway/latest/developerguide/use-custom-authorizer.html).

## HERE Maps API with Lambda Proxy
URL Mapping

|API                  | HERE URL Prefix                                 |  AWS Lambda App URL Prefix |
|-------------------- |-------------------------------------------------|-----------------------------------------------------------|
|Places               | `https://places.ls.hereapi.com/`                |  `https://<apigw>.execute-api.<region>.amazonaws.com/Prod/places/api/` |

* An example of an HTTP GET request to HERE Places API & equivalent AWS Lambda Proxy:

    __HERE Places API:__

    `https://places.ls.hereapi.com/places/v1/autosuggest?apiKey=<apiKey>&at=40.74917,-73.98529&q=chrysler`

    To call the Lambda proxy instead, replace the original URL with the API Gateway URL, change resourcePath and Query String Parameters as follows:

    __Equivalent AWS Lambda Proxy for HERE Places API:__

    API Gateway URL format:

    `https://<apigw>.execute-api.<region>.amazonaws.com/Prod/places/api/{resourcePath+}`

    {resourcePath+}: `places/v1/autosuggest?at=40.74917,-73.98529&q=chrysler`

    API Gateway URL:

    `https://<apigw>.execute-api.<region>.amazonaws.com/Prod/places/api/places/v1/autosuggest?at=40.74917,-73.98529&q=chrysler`

For details please refer [HERE Places API](https://developer.here.com/documentation/places/topics/introduction.html)

## License

Copyright (c) 2017-2019 HERE Europe B.V.

See the [LICENSE](./LICENSE) file in the root of this project for license details.
