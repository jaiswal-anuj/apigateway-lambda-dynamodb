# AWS Service Performance Optimization: Integrating API Gateway, Lambda and DynamoDB


## Architecture Diagram

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Serverless%20screen.png)



## What is Serverless?
Serverless is a cloud-native development model that allows developers to build and run applications without having to manage servers A serverless provider allows users to write and deploy code without the hassle of worrying about the underlying infrastructure. This model enhances scalability, increases developer productivity and helps to lower costs, as you only pay for the exact resources used during execution


### Details of AWS Serverless Services used in this architecture
### •	Amazon API Gateway 
Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. 

### •	AWS Lambda
AWS Lambda is a compute service that can be used to run code without provisioning or managing servers. It runs code on a high-availability compute infrastructure, operates and maintains all of the compute resources languages and integrates seamlessly with other AWS services.

### • DynamoDB
DynamoDB a cloud-based, serverless, NoSQL database service that allows users to store and retrieve data at any scale

### Purpose:
Goal is to set up a serverless infrastructure using some key AWS services and run load tests in Postman by using CRUD operations to see how the system handles increased traffic and figure out how we can fine-tuning resources to enhance its performance(Avg. response time).

We create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

•	Create, update, and delete an item.

•	Read an item.

•	Scan an item.

•	Other operations (echo, ping), not related to DynamoDB, that you can use for testing.


## Setup
Make sure all the AWS resource are created in the same region.

### Step 1: Create an IAM Role with custom policy.

Go to IAM in the AWS Console, Click Policies on the left hand panel, create policy button on the top right hand corner. Click on JSON next to Visual button on the top right hand corner in the Specify permissions screen. The Policy editor opens up, copy and paste the following policies. Click Next button and give its name as 'lambda-apigateway-policy' and then click Create policy.
Once the 'lambda-apigateway-policy' is created, let’s create a role by clicking Role on the left hand and click Create role on the top right hand corner. In the Select trusted entity page, select AWS service, choose lambda under the Use case and click Next. In the Add permission, type 'lambda-apigateway-policy' in the search box, the policy you just created will pop up, select it by checking the box and click Next. In the next page, type role name as 'lambda-apigateway-role' and click Create role. If you click Roles and type lambda in the search box, you should see your 'lambda-apigateway-role'. Go to permissions policies and expand 'lambda-apigateway-policy', you can see the below details in JSON format.

   {
   
      "Version": "2012-10-17",
      
      "Statement": [
      {
        "Sid": "Stmt1428341300017",
        "Action": [
	
            "dynamodb:DeleteItem",
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:Query",
          "dynamodb:Scan",
          "dynamodb:UpdateItem"
        ],
        "Effect": "Allow",
        "Resource": "*"
      },
      {
        "Sid": "",
        "Resource": "*",
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Effect": "Allow"
      }
      ]
   }


### Step 2: Create Lambda Function
Go AWS Lambda Console, Dashboard and click Create function.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-Lambda.png)


In the Create function page, select Author from scratch and give function name as ‘LambdaFunctionOverHttps’. Choose runtime as Python 3.12 or the latest version, leave the architecture as default as x86_64, configure the Permission by selecting the ‘Use an existing role’ that you created earlier which is 'lambda-apigateway-role' and Click Create function.
Once 'LambdaFunctionOverHttps’ got created, Function overview page for 'LambdaFunctionOverHttps' will show up as below:

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-Lambda2.png)


1. Replace the default lambda code with the following code by copying and pasting the same code editor.

#### Example Python Code

{

from _future_ import print_function

import boto3

import json

print('Loading function')
def lambda_handler(event, context):
    '''Provide an event that contains the following keys:
      - operation: one of the operations in the operations dict below      
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))

}
	
3.	Now its time to test the 'LambdaFunctionOverHttps' function that we just created. Click on the Test button right below the Deploy button and Create new test event option shows up, enter Event Name as echotest ,copy and past the event json below as well and click Save button.


{

    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-Lambda3.png)


Once the test event is created, click Test button and ‘echotest’ from select test event drop-down as shown in the diagram below, click on it. It will execute the test event and the result will show up right underneath the code editor.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Test-Lambda.png)




### Step 3: Create DynamoDB Table
Go to AWS DynamoDB console, click Create table. In the Create table page, give it Table name as 'lambda-apigateway' and partition key as id of type String as shown in the screen then click Create table button. 

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-DynamoDB.png)




### Step 4: Create API
Go to the AWS API Gateway Console and click on Create API and under the REST API click Build button.

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-API.png)


Enter API Name as 'DynamoDBOperations' and click ‘Create API’ button


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/assets/Create-API2.png)


Goto APIs Window,select and click ‘DynamoDBOperations’. In the Resources page, click Create resource button as shown in the below.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-API-Resource.png)


And type the resource name as 'DynamoDBManager' and click Create resource button.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-API-Resource2.png)


In the Resource page, expand the resource, select the resource / DynamoDBManager and click on the Create method button on the right.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-API-Resource-method1.png)


Configure the Method details as shown below. The method of this API will be POST which will call the ‘LambdaFunctionOverHttps’  i.e. Lambda Function. 
Make sure you have selected the region where you have created the 'LambdaFunctionOverHttps’  and the click Create method button.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Create-API-Resource-method2.png)


The method POST will show up as shown in the screen below. Now 'DynamoDBOperations’ ready with a / DynamoDBManager  resource with POST method
and we are ready deploy it in an environment such as dev, test, prod. Click Deploy API button and fill API Deployment configuration details
where you configure the deployment settings as shown below.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Deploy-API.png)


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/API-Stage.png)


Once the API is deployed, select the Stages on the left side and select the POST method as shown in the screenshot below. You will see Invoke URL
and that's the URL its client or consumer application will use to essentially user our 'DynamoDBOperations’ . Now copy that URL and head over to Postman,
a REST API testing application that you can download for free.

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/API-Stage-URL.png)


To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity. 

To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". 
API should execute and return "HTTPStatusCode" 200.


 {
   
    "operation": "create",
    
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}

Once you configure POST order call with the endpoint URL you copied from the API and payload above, click the Send button on top right corner. If everything goes well, it will call the 'DynamoDBOperations' --> 'LambdaFunctionOverHttps' --> 'lambda-apigateway' and it will return a standard 200 OK response as shown in the picture below. 


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Postman.png)


### Step 5: LoadTest

1.	Create a collection in Postman to organize your load test requests.
2.	Create a request with list action as shown below.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Load-Test1.png)

   
3.	Click on 3 dots next to the Load Test Collection in the Postman and select Run collection.


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Load-Test2.png)


4.	Click on Performance and configure parameters as shown below and click Run.
   

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Load-Test-Config.png)


5.	Load Test Result: Run 1 with default memory of 128 MB in 'LambdaFunctionOverHttps'
   

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Lambda-General-Config.png)


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/LoadTestRun1-Result.png)


Once the 1st Load Test completes, observe statistics such as number of requests, Avg. response time. Then add more memory in 'LambdaFunctionOverHttps'
to 1024 MB and rerun the test with the same parameters as in the 1st run.

![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/Lambda-Config-Update.png)


Load Test Result: Run 2 with default memory of 1024 MB in 'LambdaFunctionOverHttps'


![image](https://github.com/jaiswal-anuj/apigateway-lambda-dynamodb/blob/main/assets/LoadTestRun2-Result.png)


Once the 2nd Load Test completes, observe statistics such as number of requests, Avg. response time and compare with the 1st run.
By increasing Lambda's memory allocation from 128 MB to 1024 MB, the average response time improved drastically, processing more 
requests in less time without any errors. This demonstrates how fine-tuning resources can enhance scalability and performance for serverless workloads.

## 🔍 Key Takeaways:

1. Increasing the memory of a Lambda function boosts performance, making response times faster and allowing the function to handle more requests,but it also elevate costs, as AWS charges based on memory and execution time
2. It’s all about testing different setups and finding the best cost-to-performance ratio for your application.
3. Trade-off is finding the right balance- using more memory when needed to keep things efficient, but saving on costs for simpler tasks.



