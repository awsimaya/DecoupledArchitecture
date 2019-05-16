# Overview
Create a fully decoupled, microservices architecture based Leave Management application using a variety of AWS Services such as, 
* Amazon ECS
* Amazon Elastic Container Registry
* AWS Lambda
* Amazon Elastic Load Balancer
* AWS X-Ray
* AWS Step Functions
* Amazon DynamoDB
* Amazon Simple Notification Service (SNS)
* Amazon API Gateway
* AWS CloudWatch

# Application Architecture
![System Architecture](/images/DecoupledArchitectureDiagram.png)
## The application supports the following functionalities

* Submit Leave Request
* Approve / Reject Leave Request
* List Leave Requests
* Notify status to target audience

## Following are the major parts of the application

* Web front end - Serves the user interface 
* Microservices  - Serves business functionalities 
* WorkFlow services - Performs back-end business operations

### The following diagram shows the request flow between the components for a new leave request workflow

![Leave Request Workflow](/images/DecoupledArchitectureDiagram-Workflow.png)

## Setup Instructions
#### NOTE
> This is a Level 200-300 workshop and the expectation is that you are already aware of using the AWS console, basic AWS and cloud concepts such as Cloudformation, Serverless applications, API management, Load Balancers, Workflows, .NET development & Visual Studio. 

### Prerequisites / Environment setup
> I will use US-EAST-1 AWS region for all instructions here. Replace it with an appropriate region if you are using one other than US-EAST-1

* Install Visual Studio 2019 or 2017 Community edition. 
    *  Download from here - https://visualstudio.microsoft.com/vs/community/ 
* Install and configure AWS Tools for Visual Studio.
    *  Download from here - https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.AWSToolkitforVisualStudio2017
* Install and configure AWS Tools for PowershellCore.
    *  Download and configure from here - 
https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-set-up-windows.html
* Clone this GitHub repo
    * https://github.com/awsimaya/DecoupledArchitecture
### Create Workflow services
#### Create Amazon SNS Topic

> This SNS topic is used to notify users of Leave status changes

* Login to AWS console and navigate to **Amazon SNS** topic
* Enter **LeaveApproval** in the **MyTopic** textbox. Click **Next step**
* On the next page, click **Create topic**
* Copy and paste the **ARN** of the newly created SNS topic into a text editor
* Click on **LeaveApproval** link
* Under **Subscriptions**, click **Create subscription**
* Select **Email** under **Protocol** dropdown
* Enter your email address and click **Create subscription**
* You will receive a new email from **AWS Notifications** containing a link to confirm the subscription. Click **Confirm Subscription** link on the email

#### Create UpdatePayRoll Lambda Function
> This is a mock Lambda function that mimics a fake Payroll system update

* Open Visual Studio
* Open UpdatePayRoll.sln solution from UpdatePayRoll folder in this repo
* Right click on the project file and choose **Publish to AWS Lambda**
* In the wizard that pops up, choose a region you want the solution to be deployed to and click **Next**
* Select **AWSLambdaFullAccess** role for the **Role Name** dropdown and click **Upload**
* Your Lambda application is now deployed to your AWS environment
* Navigate to the newly created Lambda function home page on the AWS console
* Copy the ARN for Lambda from the top right of the screen and paste into a text editor

#### Create UpdateHRSystem Lambda Function

> This is a mock Lambda function that mimics a fake HR system update

* Open Visual Studio
* Open **UpdateHRSystem.sln** solution from **UpdateHRSystem** folder in this repo
* Right click on the project file and choose **Publish to AWS Lambda**
* In the wizard that pops up, choose a region you want the solution to be deployed to and click **Next**
* Select **AWSLambdaFullAccess** role for the **Role Name** dropdown and click **Upload**
* Your Lambda application is now deployed to your AWS environment
* Navigate to the newly created Lambda function home page on the AWS console
* Copy the ARN for Lambda from the top right of the screen and paste into a text editor

#### Create LeaveApprovalWorkflow Step Function
* Open **workflow.json** from **StepFunctionWorkFlow** folder
* Replace **<LAMBDA_ARN_FOR_UpdateHRSystem>** with the ARN you saved into the text editor in **Create UpdateHRSystem Lambda Function** step
* Replace **<LAMBDA_ARN_FOR_UpdatePayRoll>** with the ARN you saved into the text 
editor in **Create UpdatePayRoll Lambda Function** step
* Replace **<SNS_TOPIC_ARN>** with the ARN you saved into the text editor in **Create Amazon SNS Topic** step
* Login to AWS console
* Navigate to AWS Step Functions home page by typing **Step Functions** on the search bar and selecting the first result
* Ensure **Author with code snippets** option is selected. Enter **LeaveApprovalWorkflow** in the **NAME** textbox
* Copy all the content from **workflow.json** file and paste it into the **State machine definition** textbox
* Ensure **Create an IAM role for me** option is selected. Enter **leaveapprovalworkflowrole** in the **Name** textbox
* Click **Create state machine**
* Your step function is created successfully
* Copy and paste the ARN of the Step Function you just created to a text editor for later usage

### Create Microservices

>In this section we will create the Lambda functions used in processing requests from the web front end, API Gateway endpoints and a DynamoDB table to store Leave data

#### Create LeaveManagementAPIs
We will be creating the following resources as a result of this section

 * A DynanmoDB table named **LeaveRequests**
 * A Lambda function called **AddOrUpdateLeaveRequest**
 * A Lambda function called **GetLeaveRequests**
 * An API Gateway API called **LeaveManagementAPI**

#### Steps

* Open **LeaveManagementAPI.sln** from **LeaveManagementAPI** folder
* Right click on **LeaveManagementAPI** project and click **Publish to AWS Lambda...**
* On the newly opened wizard, Enter **LeaveManagementAPI** in the **Stack Name** textbox
* Ensure **LeaveRequests** is the value of the **LeaveTableName** textbox
* Select **true** for **ShouldCreateTable** textbox
* Click **Publish**
* Visual Studio will open a new screen called **Stack: LeaveManagementAPI** to show the events taking place in creating the stack
* Once you see **CREATE_COMPLETE** in green color, copy the URL from **AWS Serverless URL** and paste into a text editor
* Navigate to DynamoDB home page on AWS console and select **LeaveRequests** table under **Tables**
* Copy the **Latest stream ARN** value (shown in the picture below) and paste it into a text editor for later usage

![DynamoDB setting](/images/DynamoDB.png)

* Navigate to API Gateway home page on the AWS console and select **LeaveManagementAPI** API
* Select **Stages** and select **Prod** 
* Under **Logs/Tracing** tab, check **Enable CloudWatch Logs** and **Enable X-Ray Tracing** checkboxes as shown below

![API Gateway XRay setting](/images/APIGAtewayXRay.png)

* Open **LeaveRequestProcessor.sln** solution under **LeaveRequestProcessor** folder
* In **Function.cs** file, replace **<SNS_TOPIC_ARN>** with the ARN you saved into the text editor in **Create Amazon SNS Topic** step
* In **Function.cs** file, replace **<STEP_FUNCTION_STATEMACHINE_ARN>** with the ARN you saved into the text editor in **Create LeaveApprovalWorkflow Step Function** step
* In **serverless.template** file, replace **<DYNAMO_DB_STREAM_ARN>** with the DynamoDB stream ARN you created earlier
* Right click on **LeaveRequestProcessor** project and select **Publish to AWS Lambda...**
* In the publish wizard, enter **LeaveRequestProcessor** in the **Stack Name** textbox
* Enter any name for the S3bucket in the **S3 Bucket** text box
* Click **Publish**
* Visual Studio will open a new window to show the progress of the deployment and will show **CREATE_COMPLETE** once successfully deployed