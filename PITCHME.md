### Production-ready Serverless

---

### What you will possibly learn

- Getting started with serverless
- How production-ready serverless system looks
- CI & Deployment
- Approaches to test serverless solution
- Monitoring & Troubleshooting

Note:
Show experience with aws console
What makes your lambda production ready: logging, exception handling

What it takes to build successful serverless applications: production-ready code, automation and monitoring
Talk about lambda function naming convention and best practices (prefixes, dash separator etc.)
Remember to explain debugging: serverless offline, unit test, SAM, localstack etc.



Hi, I am ...
...

We are going to share our experience with serverless and the steps required to get it done right.
Serverless is extremely popular these days and everyone wants to try it, but it's problematic.
It's like trying to learn how to swim and start it in the ocean.
The real problem is the documentation: it's enormous, it's huge and you will spend many days reading it
and trying to figure out how to start. There are so many options and flavors.
We do not focus on what the serverless is, however they do not mentions key things:
project structure, naming convention, troubleshooting etc.

---

### Getting started

- Install NodeJs
- Install VS Code
- Create new git repository

Notes:
How do you start? Create a repository, generate project, but wait... What is the structure of project?
---

#### Setup dev environment

- Serverless framework
- AWS SAM local
- LocalStack from Atlassian

---

#### Serverless framework

```bash
# Installing the serverless cli
npm install -g serverless

#Setup credentials
serverless config credentials --provider aws --key <access key> --secret <secret key>

# Create a new Serverless Service/Project
$ serverless create --template aws-nodejs --path <replace with service name>
```

---

#### Serverless framework: result

# Invoke the Function locally
serverless invoke -f <replace with function name> -l

TODO: show result

Notes:
Why Serverless framework? It gets an overview of all resources: api endpoints, lambda, s3 bucket, sns etc.
Serverless generates a project with function that can be imnvoked, still need to define debug configuration

---

#### SAM Local

```bash
npm install -g aws-sam-local

# Generate sample event source
sam local generate-event api
```

Notes:

# Invoke a function locally in debug mode on port 5858
$ sam local invoke <function logical id> -e event.json --log-file ./output.log

```
#Test API endpoint
curl http://localhost:3000/products

# Run API Gateway locally
sam local start-api

# Package SAM template
$ sam package --template-file sam.yaml --s3-bucket mybucket --output-template-file packaged.yaml

# Deploy packaged SAM template
$ sam deploy --template-file ./packaged.yaml --stack-name mystack --capabilities CAPABILITY_IAM
```

---

#### AWS LocalStack

- API Gateway at http://localhost:4567
- Lambda at http://localhost:4574
- CloudWatch at http://localhost:4582

```bash
aws kinesis list-streams --endpoint-url=http://localhost:4568
```

Notes: 
Simulates cloud environment by spinning up core Cloud APIs available via custom http endpoints on localhost. Example:
You can use aws cli commands by instructing it to use local infrastucture:
See more: https://github.com/localstack/localstack


Difficult to get it working, better to use for more advanced scenarios such as email sending and end-to-end testing

---

#### Setup prod environment

- Create IAM user(s) per lambda
- Create S3 bucket
- Define access policies

TODO: show screenshot

Notes:

Permissions from IT point of view
Use existing resources rather than generate new
1000 of policies for different actions
Use CloudTrail to audit changes

---

#### Project structure

TODO: naming conventions: file with function and function handler, prefix with project to search in AWS Console
TODO: show long list of unprefixed functions

TODO: show the bad structure with lambda functions in single folder

TODO: show the good structure with src and lambda functions grouped per folder, event.json, config, tests etc.

---

#### How to debug

TODO: set the debug in vscode
TODO: debug in npm

---

### Recap

TODO: summarize topics
TODO: setup Kahoot to check the knowledge

---

### Making production-ready code

TODO: ping endpoint for lambda to return the build version (with api gateway only). Otherwise where to get the version of lambda

Notes:
Project structure is defined, functions can be invoked locally, it's time to make them production-ready and deploy them
Add exception handling, logging, built-in retry feature, dead-letter queue with CloudWatch/NewRelic alert

---

### Logging: basic

```javascript
module.exports.handler = (event, context, callback) => {
  console.log('Running event');

    SendSMS(event.to, 'Hello from Lambda Functions!', (err, res) => {
        if (err) callback(err);

        const response = {
            statusCode: res.statusCode,
            body: JSON.stringify({
                message: `Successfully sent sms to ${event.to}.`,
            });
        };

        //signal success
        callback(null, response);
    });
};
```

@[4](Log event data)
@[6](Wrap in try/catch.)
@[7](Log response data)

Note:
Use NodeJs for simplicity
hello-world vs logging with log levels, retry and exception-handling

[comment]: <> (TODO: Show old and new side by side, check pitchme how to do)


### Logging: production-ready

```javascript
'use strict';

const log4js = require('log4js');
const logger = log4js.getLogger('server');


module.exports.handler = (event, context, callback) => {
  logger.level = process.env.LOG_LEVEL;
  logger.addContext('name', 'Sms Sender');
 
  logger.info(`Running event ${JSON.stringify(event)}`);

    try{
        logger.debug(`Sending sms message to ${event.to}...`);

        SendSMS(event.to, 'Hello from Lambda Functions!', (err,res) => {
            logger.debug(`Sent sms message to ${event.to}.`);

            if (err) {
                console.error(`Received an error upon sending sms message to ${event.to}. Error: ${err}`);
                callback(err);
            }

            logger.info(`Successfully sent sms message to ${event.to}. Response: ${JSON.stringify(res)}`);
            const response = {
                statusCode: res.statusCode,
                body: JSON.stringify({
                    message: `Successfully sent sms to ${event.to}.`,
                });
            };

            logger.debug(`Done, calling callback`);

            //signal success
            callback(null, response);
        });
    }
    catch(error){
        logger.error(`Failed to send the sms message to ${event.to}. Error: ${error}`);
        callback(err);
    }
};
```

Note:
Show how to find CloudWatch logs
Mention that CloudWatch log entries have requestId (context.awsRequestId) that can be used as corellation id

---

### CI & Deployment

TODO: show screenshot from TeamCity, build version in lambda 
TODO: CloudFormation
TODO: aws cli

Notes:
Use aws tools vs existing tools such as TeamCity and Octopus
Deployment with aws cli/powershell vs Serverless vs CloudFormation scripts vs Terraform

---

### Testing

---

#### Unit test

TODO: stubbing http library
TODO: show screenshots with tests and package.json

---

#### Integration tests

TODO: show how to write tests in javascript
TODO: share Postman collection and newman, custom args
TODO: e2e test to invoke lambda and read logs

---

### Monitoring & Troubleshooting

TODO: use CloudWatch
TODO: use NewRelic

---

### Summary

TODO: summarize what is covered

---