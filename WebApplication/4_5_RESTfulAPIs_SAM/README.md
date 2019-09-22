# Module 4.5: Using SAM and the SAM CLI

In this module you will rebuild everything you built in modules 2-4 utilizing SAM and the SAM CLI. By the end of this module you will have a second API, lambda, and Cognito Userpool implementation.

## Implementation Instructions

### 1. Setting up the environment
Before we create our new application, there are several one-time steps we need to do to setup our development environment. Run these steps from within a terminal in your Cloud9 web IDE. 

**:white_check_mark: Step-by-step directions**
1. Get the latest AWS-CLI and AWS-SAM-CLI
    ```
    pip install --upgrade --user awscli aws-sam-cli
    ```
1. Create an S3 bucket for storing our deployment (the bucket must be a unique name)
    ```bash
    aws s3 mb s3://<unique-bucket-name>
    ```

### 2. Create a SAM Application
Use the SAM CLI to create a new serverless application for the backend. Run these steps from within a terminal in your Cloud9 web IDE.

**:white_check_mark: Step-by-step directions**
1. Use sam init to create the new application
    ```bash
    sam init -n wr-sam
    ```
1. Change to the new *wr-sam* directory
    ```bash
    cd wr-sam
    ```
1. Rename the hello-world folder
    ```bash
    mv hello-world request-unicorn
    ```

### 4. Update the SAM template file
The SASM template, located in the root and called **template.yaml** is the full description of the new application. This needs to be updated from [template-sam.yaml](template-sam.yaml)

**:white_check_mark: Step-by-step directions**
1. Open **wr-sam** > **template.yaml**
1. Replace the entire contents with the text from [template-sam.yaml](template-sam.yaml)

### 4. Update the Lambda Function code
Update Lambda function application code with [requestUnicorn.js](requestUnicorn.js)

**:white_check_mark: Step-by-step directions**
1. Open **wr-sam** > **request-unicorn** > **app.js**
1. Replace the entire contents with the code from [requestUnicorn.js](requestUnicorn.js)

### 5. Package and deploy the application
Deploy the serverless application to your account. 

**:white_check_mark: Step-by-step directions**
1. Build the application to remove any unneeded files. Run these steps from within a terminal in your Cloud9 web IDE from the *wr-sam* root.
    ```bash
    sam build
    ```
1. Package the application and upload it to the bucket created in step 1. (Setting up the environment)
    ```bash
    sam package --s3-bucket <replace with your bucket-name> --output-template out.yaml
    ```
1. Deploy the application
    ```bash
    sam deploy --template-file out.yaml --stack-name wild-rydes --capabilities CAPABILITY_IAM
    ```

### 6. Update the Website Config
Update the /js/config.js file in your website deployment to update to the new application params that you just created. The parameters have been output to the CloudFormation stack.

**:white_check_mark: Step-by-step directions**
1. Open up the [CloudFormation console](cloudformation-console)
1. Choose *wild-rydes* from the stack list
1. Choose the **Outputs** tab
1. On your Cloud9 development environment open `js/config.js`
1. Update the **userPooliId**, **userPoolClientId**, **region**, and **invokeUrl** setting in the config.js file from the values on the **Outputs** tab.
    An example of a complete `config.js` file is included below. Note, the actual values in your file will be different.
    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
            region: 'us-west-2' // e.g. us-east-2
        },
        api: {
            invokeUrl: 'https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod,
        }
    };
    ```

1. Save the modified file making sure the filename is still `config.js`.
1. Commit the changes to your git repository:
    ```
    $ git commit -am "update to new api"
    $ git push
    ```

    [Amplify Console][amplify-console-console] should pick up the changes and begin building and deploying your web application. Watch it to verify the completion of the deployment.


## Implementation Validation
You will need to register your email again as this is using a new UserPool in cognito.

**:white_check_mark: Step-by-step directions**
1. Go to `/register.html` to register
1. On verification email go to `/verify.html` to verify email
1. Login to your site at `/signin.html`
1. Test requesting a unicorn as before.

## Extra Credit: Test Locally
With SAM CLI you can test Lambda functions locally. We need to create several files to do so. First, we will create an event.json file that contains a post event as if we were hitting it with an API endpoint. Second we will create a local environent file that tells the Lambda what values it should use for environment variables locally.

**:white_check_mark: Step-by-step directions**
1. Create a file in the root of **wr-sam** called *env.json*
1. Copy the entire content of [env-sam.json](/env-sam.json) and paste it to *env.json*
1. Update the Table Name: Your table name value is found on the [CloudFormation Console](cloudformation-console) under the **wild-rydes** stack on the **Outputs** tab.
1. Create a file in the root of **wr-sam** called *event.json*
1. Copy the following in to the *event.json* file
    ```json
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```
1. In the root of the **wr-sam** folder in the terminal run the following command
    ```bash
    sam local invoke -e event.json -n env.json
    ```
You should see a response similar to this:
```bash
Invoking app.lambdaHandler (nodejs10.x)
2019-09-22 00:14:28 Found credentials in shared credentials file: ~/.aws/credentials
arn:aws:lambda:eu-central-1:700336187521:layer:aws-sdk:5 is already cached. Skipping downloadRequested to skip pulling images ...

Mounting /home/ec2-user/environment/wr-sam/.aws-sam/build/LambdaFunction as /var/task:ro,delegated inside runtime container
START RequestId: c109149a-5417-13c6-6791-2616175999c3 Version: $LATEST
   '{"PickupLocation":{"Latitude":47.6174755835663,"Longitude":-122.28837066650185}}' }d event ( IWPt8JIRtjRx_rCn2l-AVw ):  { path: '/ride',
2019-09-22T00:14:30.143Z        c109149a-5417-13c6-6791-2616175999c3    INFO    Finding unicorn for  47.6174755835663 ,  -122.28837066650185
END RequestId: c109149a-5417-13c6-6791-2616175999c3
REPORT RequestId: c109149a-5417-13c6-6791-2616175999c3  Duration: 103.43 ms     Billed Duration: 200 ms Memory Size: 128 MB     Max Memory Used: 56 MB
{"statusCode":201,"body":"{\"RideId\":\"IWPt8JIRtjRx_rCn2l-AVw\",\"Unicorn\":{\"Name\":\"Bucephalus\",\"Color\":\"Golden\",\"Gender\":\"Male\"},\"UnicornName\":\"Bucephalus\",\"Eta\":\"30 seconds\",\"Rider\":\"the_username\"}","headers":{"Access-Control-Allow-Origin":"*"}}
```

### :star: Recap

:key: You have now built the WildRydes backend from a SAM template that is easily editable and can be versioned.

:wrench: You also use SAM CLI to locally test tha Lambda.

:star: Congratulations, you have completed the Wild Rydes Web Application Workshop! Check out our [other workshops](../../README.md#workshops) covering additional serverless use cases.

### Next

:white_check_mark: See this workshop's [cleanup guide][cleanup] for instructions on how to delete the resources you've created.


[cloudformation-console]: https://console.aws.amazon.com/cloudformation/home
[cleanup]: ../9_CleanUp/