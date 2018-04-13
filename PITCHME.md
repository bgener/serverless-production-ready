### Production-ready Serverless
 
---

### What you will learn

- Setup local environment
- Logging
- Debugging
- Deployment
- Permissions
- Monitoring
- Testing
- Troubleshooting

Note:
Show experience with aws console
What makes your lambda production ready: logging, exception handling

What it takes to build successful serverless applications: production-ready code, automation and monitoring
Talk about lambda function naming convention and best practices (prefixes, dash separator etc.)
Remember to explain debugging: serverless offline, unit test, SAM, localstack etc.

---

### Setup local environment

#### Using Serverless framework

- Install VS Code
- Install Serverless framework

```bash
# Installing the serverless cli
npm install -g serverless

# Create a new Serverless Service/Project
$ serverless create --template aws-nodejs --path my-service

# Invoke the Function
serverless invoke -f hello -l

# Deploy the Service
serverless deploy -v

# Remove the deployed Service
serverless remove
```

#### Using AWS SAM

- Install Docker
- Install sam-local

The following will install SAM Local:

```bash
npm install -g aws-sam-local

# Generate sample event source
sam local generate-event api

# Run API Gateway locally
sam local start-api

# Invoke a function locally in debug mode on port 5858
$ sam local invoke <function logical id> -e event.json --log-file ./output.log

#Test API endpoint
curl http://localhost:3000/products

# Package SAM template
$ sam package --template-file sam.yaml --s3-bucket mybucket --output-template-file packaged.yaml

# Deploy packaged SAM template
$ sam deploy --template-file ./packaged.yaml --stack-name mystack --capabilities CAPABILITY_IAM
```

#### Using AWS localstack

Simulates cloud environment by spinning up core Cloud APIs available via custom http endpoints on localhost. Example:

- API Gateway at http://localhost:4567
- Lambda at http://localhost:4574
- CloudWatch at http://localhost:4582

You can use aws cli commands by instructing it to use local infrastucture:

```bash
aws kinesis list-streams --endpoint-url=http://localhost:4568
```

See more: https://github.com/localstack/localstack

---

### Logging: basic

```javascript
'use strict';

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

module.exports.handler = (event, context, callback) => {
  console.info(`Running event ${JSON.stringify(event)}`);

    try{
        console.debug(`Sending sms message to ${event.to}...`);

        SendSMS(event.to, 'Hello from Lambda Functions!', (err,res) => {
            console.debug(`Sent sms message to ${event.to}.`);

            if (err) {
                console.error(`Received an error upon sending sms message to ${event.to}. Error: ${err}`);
                callback(err);
            }

            console.info(`Successfully sent sms message to ${event.to}. Response: ${JSON.stringify(res)}`);
            const response = {
                statusCode: res.statusCode,
                body: JSON.stringify({
                    message: `Successfully sent sms to ${event.to}.`,
                });
            };

            console.debug(`Done`);

            //signal success
            callback(null, response);
        });
    }
    catch(error){
        console.error(`Failed to send the sms message to ${event.to}. Error: ${error}`);
        callback(err);
    }
};
```

Note:
CloudWatch log entries have requestId (context.awsRequestId)

---