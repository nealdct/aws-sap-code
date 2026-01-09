## PART 1 - SQS - Lambda - DynamoDB Table ##

Region: us-east-1

Note your AWS account ID: *ACCOUNT_ID*

1. Create DynamoDB Table:
	
Name: ProductVisits
Partition key: ProductVisitKey
	
2. Create SQS Queue:

Name: ProductVisitsDataQueue
Type: Standard
	
Note the Queue URL: *QUEUE URL*

3. Go to AWS Lambda and create function:
	
Name: productVisitsDataHandler
Runtime: Node.js 20.x
Role: create new role from templates
Role name: lambdaRoleForSQSPermissions
Add policy templates: "Simple microservice permissions" and "Amazon SQS poller permissions"
	
4. From actions menu in front of function code heading upload a zip file (DCTProductVisitsTracking.zip)
5. Go back to SQS and open "ProductVisitsDataQueue"
6. Configure Lambda function trigger and specify Lambda function:

Name: productVisitsDataHandler

7. Go to AWS CLI and send messages:

AWS CLI Command: `aws sqs send-message --queue-url *QUEUE URL* --message-body file://message-body-1.json`
Modify: Queue name and file name
File location: Code/build-a-serverless-app/part-1

## PART 2 - DynamoDB Streams - Lambda - S3 Data Lake ##

1. Go to DynamoDB table
2. On the "Exports and streams" configuration tab, enable a DynamoDB stream for "New Image"
3. Create S3 bucket in same region:

Name: product-visits-datalake
Modify: bucket name by adding letters/numbers at end to be unique
Region: us-east-1

4. Go to IAM and create a policy:

Name: productVisitsLoadingLambdaPolicy
JSON: Copy contents of "lambda-policy.json"
Modify: Replace account number / region / names as required

5. Create a role:

Use case: Lambda
Policy: productVisitsLoadingLambdaPolicy
Name: productVisitsLoadingLambdaRole

6. Unzip "DCTProductVisitsDataLake.zip" 
7. Edit index.js and update bucket name entry:

Bucket: 'product-visits-datalake'
Note: Change bucket name to YOUR bucket name

8. Then zip up contents (don't zip the whole folder) into "DCTProductVisitsDataLake.zip"
9. Create a function:

Name: productVisitsDatalakeLoadingHandler
Runtime: Node.js 20.x
Role: productVisitsLoadingLambdaRole
	
10. Upload the code: DCTProductVisitsDataLake.zip
11. Go to DynamoDB and configure the table
12. Choose "Export and streams", and under "DynamoDB stream detils, create a trigger
13. Select function:

Name: productVisitsDatalakeLoadingHandler

14. Go to AWS CLI and send messages:

AWS CLI Command: `aws sqs send-message --queue-url *QUEUE URL* --message-body file://message-body-1.json`
Modify: Queue name and file name
File location: Code/build-a-serverless-app/part-2

## PART 3 - S3 Static Website - API Gateway REST API - Lambda ##

1. Create IAM Policy:

JSON: Copy from lambda-policy.json
Updates: change account number
Name: productVisitsSendMessageLambdaPolicy

2. Create an IAM role:

Use case: Lambda
Policy: productVisitsSendMessageLambdaPolicy
Name: productVisitsSendMessageLambdaRole
	
3. Unzip "DCTProductVisitForm.zip"
4. Edit index.js for backend and update queue name:

QueueUrl: "*QUEUE URL*"

Note: change above URL to YOUR queue URL

5. Then zip the backend folder contents to backend.zip
6. Create a Lambda function:

Name: productVisitsSendDataToQueue
Runtime: Node.js 20.x
Role: productVisitsSendMessageLambdaRole
	
7. Upload code: backend.zip
8. Go to Amazon API Gateway
9. Create a REST API and select New API:

Name: productVisit
Endpoint type: Regional
	
10. Create a resource:

Resource name: productVisit
Resource path: /productvisit
Enable CORS
	
11. Create a method:

Type: PUT
Integration type: Lambda function
Use Lambda Proxy Integration
Function: productVisitsSendDataToQueue
	
12. Deploy API - Actions > Deploy API
13. Create a new stage called "dev"
14. Go to SDK Generation and generate using platform "JavaScript"
15. Unzip the file, change into the extract folder, and copy the file contents into the frontent folder
16. Create a bucket:

Name: product-visits-webform
Updates: Add letters/numbers to bucket name to be unique
Region: us-east-1
Turn off block public access
    
17. Enable static website hosting:

Index: index.html
Policy: copy contents of frontend-bucket-policy.json (edit bucket name)
	
18. Edit index.html with correct Region if required:

region: 'us-east-1' // set this to the region you are running in.

19. Use command line to change to folder containing the frontend directory
20. Upload contents with AWS CLI command (change bucket name)

`aws s3 sync ./frontend s3://product-visits-webform`

21. Copy the object URL for index.html
22. Use URL to access application and then test submitting data using the form

## Part 4 - Create CloudTrail Trail and EventBridge Rule

Please follow the steps in the video.

## Part 5 - Query S3 Data Lake with Amazon Athena

- In the Athena Query editor go to settings and set an S3 bucket as the query result location (do this before the below steps)

1. In Athena navigate to "Data sources"
2. Create a new data source
3. Select "S3 - AWS Glue Data Catalog"
4. Select "Create a table manually"
5. Enter the following information:

- Table name: MyDataLake
- Create a database
- Database name: s3_data_lake
- Dataset: s3://product-visits-datalake/data/ ***modify the bucket name**
- Table type: Apache Hive
- File format: CSV

6. Use the "Bulk add columns" button under "Column details"
7. Copy contents of table_columns.md file and paste in window




