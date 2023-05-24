# Module 3.15 Assignment
3.15 Continuous Deployment - Secret Management

## Objective
The objective of this assignment is to set up a continuous integration and continuous deployment (CI/CD) pipeline for a Node.js application that is deployed on a serverless platform with Logging and Monitoring in placed!.

## Step 1: Create a code repository in GitHub
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/f5b7f2be-ccd2-4dc0-9ee3-7520593cbaff)

## Step 2: Clone the code repository into local machine
```
$ git clone git@github.com:liaucg/module_3.15_assignment.git
```

## Step 3: Create index.js file
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/6e66d921-d381-4292-b961-1c98513db557)

[index.js](index.js)
```js
module.exports.handler = async (event) => {
    return {
      statusCode: 200,
      body: JSON.stringify(
        {
          message: "Go Serverless v3.0! Your function executed successfully!",
          input: event,
        },
        null,
        2
      ),
    };
  };
```

## Step 4: Create serverless.yml
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/dc40b874-decf-4d82-ac77-1e6fdf6f69a2)

[serverless.yml](serverless.yml)
```yml
service: liau-module-3-15-assignment
frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x
  region: ap-southeast-1

functions:
  api:
    handler: index.handler
    events:
      - httpApi:
          path: /
          method: get

plugins:
  - serverless-offline
```

## Step 5: Deploy and verify that the serverless application is working
Install dependencies with:
```
$ npm install
```

and then deploy with:
```
$ serverless deploy
```

Following is the output from deploy command:
```
Running "serverless" from node_modules

Deploying liau-module-3-15-assignment to stage dev (ap-southeast-1)

✔ Service deployed to stack liau-module-3-15-assignment-dev (129s)

endpoint: GET - https://ntr33tfrmb.execute-api.ap-southeast-1.amazonaws.com/
functions:
  api: liau-module-3-15-assignment-dev-api (53 MB)
```

After succesful deployment the created serverless application can be invoked with curl command:
```
curl https://ntr33tfrmb.execute-api.ap-southeast-1.amazonaws.com/
```

which resulted in the following response:
```json
{
  "message": "Go Serverless v3.0! Your function executed successfully!",
  "input": {
    "version": "2.0",
    "routeKey": "GET /",
    "rawPath": "/",
    "rawQueryString": "",
    "headers": {
      "accept": "*/*",
      "content-length": "0",
      "host": "ntr33tfrmb.execute-api.ap-southeast-1.amazonaws.com",
      "user-agent": "curl/7.81.0",
      "x-amzn-trace-id": "Root=1-646e0f93-49116b9220b8106165a13de0",
      "x-forwarded-for": "118.200.182.45",
      "x-forwarded-port": "443",
      "x-forwarded-proto": "https"
    },
    "requestContext": {
      "accountId": "255945442255",
      "apiId": "ntr33tfrmb",
      "domainName": "ntr33tfrmb.execute-api.ap-southeast-1.amazonaws.com",
      "domainPrefix": "ntr33tfrmb",
      "http": {
        "method": "GET",
        "path": "/",
        "protocol": "HTTP/1.1",
        "sourceIp": "118.200.182.45",
        "userAgent": "curl/7.81.0"
      },
      "requestId": "FbdfBiopyQ0EMpQ=",
      "routeKey": "GET /",
      "stage": "$default",
      "time": "24/May/2023:13:22:27 +0000",
      "timeEpoch": 1684934547006
    },
    "isBase64Encoded": false
  }
}
```

## Step 6: Create CI/CD pipeline with GitHub Actions
Create main.yml in .github/workflows folder
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/d3a2863b-fa8c-4bd8-8830-5aca921b7dee)

[main.yml](.github/workflows/main.yml)
```yml
name: CICD for Serverless Application
run-name: ${{ github.actor }} is doing CICD for Serverless Application

on:
  push:
    branches:  [ main, "*"]

jobs:
  pre-deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."

  install-dependencies:
    runs-on: ubuntu-latest
    needs: pre-deploy
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Run Installation of Dependencies Commands
        run: npm install

  deploy:
    name: deploy
    runs-on: ubuntu-latest
    needs: install-dependencies
    strategy:
      matrix:
        node-version: [18.x]
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm ci
    - name: serverless deploy
      uses: serverless/github-action@v3.2
      with:
        args: deploy
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Workflow Syntax
**name**: The name of the workflow.

**on**: The type of event that can run the workflow. This workflow will only run when there is a git push to either the main or other branch.

**jobs**: A workflow consists of one or more jobs. Jobs run in parallel unless a ***needs*** keyword is used. Each job runs in a runner environment specified by ***runs-on***.

**steps**: A sequence of tasks to be carried out.

**uses**: Selects an action to run as part of a step in your job. An action is a reusable unit of code.

**with**: A map of input parameters.

**run**: Runs command line programs.

**env**: Set the environment variables.

## Step 7: Add AWS_ACCESS_KEY_ID and ASW_SECRET_ACCESS_KEY to GitHub Secrets
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/91282513-1aee-4273-924b-4e7d0f51cd93)

## Step 8: Push changes to GitHub to start the workflow
Commit changes locally and push it to GitHub. Navigate the repo on GitHub, click on the **Actions** tab to see the workflows.
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/91776f94-7ca8-4265-9731-e102818dbd16)

![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/fb257691-2ac4-4ff1-ab1d-31d36e24291f)

## Step 9: Add a secret in AWS Secrets Manager
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/aba829da-c0c4-46c9-83c1-ac3af2a5826b)
![image](https://github.com/liaucg/module_3.15_assignment/assets/22501900/f9b6de9c-83ff-4b5c-a983-6ac760517657)

## Step 10: Retrieve the stored secret from AWS Secrets Manager as part of the CI/CD pipeline
Add a new job **retrieve-secret** to .github/workflows/main.yml to retrieve the stored secret *LIAU_SECRET_2* from AWS Secrets Manager and inject into environment variables
```yml
retrieve-secret:
  runs-on: ubuntu-latest
  needs: install-dependencies
  steps:
  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v1
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ap-southeast-1

   - name: Read secrets from AWS Secrets Manager into environment variables
     uses: abhilash1in/aws-secrets-manager-action@v2.1.0
     with:
       secrets: LIAU_SECRET_2
       parse-json: false
```

The last step added will print the value of *LIAU_SECRET_2*
```yml
- name: Check if env variable is set after fetching secrets
  run: if [ -z ${LIAU_SECRET_2+x} ]; then echo "LIAU_SECRET_2 is unset"; else echo "LIAU_SECRET_2 is set to '$LIAU_SECRET_2'"; fi
```

**secrets**:
 - List of secret to be retrieved.
 - Examples:
   - To retrieve a single secret, use ```secrets: my_secret_1```
   - To retrieve multiple secrets, use:
     ```
     secrets: |
     my_secret_1
     my_secret_2
     ```
   - To retrieve "all secrets having names that contain dev" or "begin with app1/dev/", use:
     ```
     secrets: |
     *dev*
     app1/dev/*   
     ```
**parse-json**:
 - If parse-json: true and secret value is a valid stringified JSON object, it will be parsed and flattened. Each of the key value pairs in the flattened JSON object will become individual secrets. The original secret name will be used as a prefix.

## Step 10: Push changes to GitHub to start the workflow
Commit changes locally and push it to GitHub. Navigate the repo on GitHub, click on the **Actions** tab to see the workflows.
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/5668c788-2522-4210-a988-f2f4f9950c9e)
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/54a2122b-56ae-4724-aab0-95a78913c61a)

