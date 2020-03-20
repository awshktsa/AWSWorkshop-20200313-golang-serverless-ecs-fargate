# AWSTPEWorkshop-20200313-golang-serverless-ecs-fargate
For this workshop, we will have a quick review about how to deploy lambda and ecs-fargate. And this time, we will use "GO" as our target application language.
- Start from 2018, AWS Lambda support Go for developing serverless application - https://aws.amazon.com/about-aws/whats-new/2018/01/aws-lambda-supports-go/(https://aws.amazon.com/about-aws/whats-new/2018/01/aws-lambda-supports-go/)
And we work with Golang Community to have this workshop together, we will use lambda to build a "Go Hello World - Serverless", and also we will extend the infrastructure to Elastic Container Service - Fargate with "Web service Framework -Gin".
------

##In this workshop, you will learn following tools to work with AWS:
1. awscli - AWS Command Lint Tool Official Site https://aws.amazon.com/tw/cli/(https://aws.amazon.com/tw/cli/)
2. sam for lambda - What is SAM(https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
3. cdk - AWS Cloud Development Kit Official Site https://aws.amazon.com/cdk/?nc1=h_ls(https://aws.amazon.com/cdk/?nc1=h_ls)

------
### Step 1:
* Switch Region on the AWS console, a drag down menu near right-up corner.
Pick one region close to you, if you don't have any prefer, use **us-east-1**

------
### Step 2: Setup IAM Role/User for this workshop
* 2-a: For user who want to use Cloud9
- **AWS Console > Services > IAM > Role**
- Create Role, service pick "EC2" and click "Next"
- Search "AmazonS3FullAccess" and click the check box
- also search following policies and attach to this role: "AWSLambdaFullAccess","AmazonEC2ContainerRegistryFullAccess","AmazonECS_FullAccess","AWSCloudFormationFullAccess","AmazonVPCFullAccess","IAMFullAccess"
- Click Next, Inut Tag Key and Value if you want, click Next to enter "Name" and "Description" for the Role.
- Input "golang-workshop-cloud9-role" for the "Name" and click "Create".

* 2-b: For user who want to use your own laptop instead of using Cloud9
- **AWS Console > Services > IAM > User**
- Create User and attach following policy to this user: ["AmazonS3FullAccess","AWSLambdaFullAccess","AmazonEC2ContainerRegistryFullAccess","AmazonECS_FullAccess","AWSCloudFormationFullAccess","AmazonVPCFullAccess","IAMFullAccess"]
- Generate the credential "ACCESS_KEY" and "SECRET_KEY" and keep it safe, never share with anybody else and remeber to deactivate this user after workshop.
------

### Step 3: Launch a Cloud9 IDE for our workshop env (Note: if you already had a working environment in your local laptop, then please skip this step and jump to Step 3-b)
* **AWS Console > Services > Cloud9 > Create Environment**
- Enter "Name" and "Description" for your Environment, and click Next Step
- Select "Create a new instance for environment (EC2)" for Environment Type
- Select "t2.micro" for Instance Type
- Select "Amazon Linux" for Platform, and click Next Step
- Input Tag Key and Value if you want, and click "Create Environment"
------
***Then we need to Attach the IAM Role onto this Cloud9 Environment***
- **AWS Console > Services > EC2 > Instances > Click the EC2 Instance for your Cloud9 > Actions > Instance Settings > Attach/Replace IAM Role > Choose the IAM Role we created in Step2**
- **Back to Cloud9 > Cloud9 Icon (Upper Left Corner) > Perference > AWS Settings > Credential > AWS Managed Temperory Credential > Off**
------


### Step 4: Setup Go Development Environment in your Cloud9
- After the IDE launch, click to "bash" tab in the botton of the IDE page
- We will follow the AWS Cloud9 User Guide to setup our Go env - https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-go.html(https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-go.html) has all the detail, and we copy the mendatory shell command in below:

```
sudo yum -y update
wget https://storage.googleapis.com/golang/go1.9.3.linux-amd64.tar.gz # Download the Go installer.
sudo tar -C /usr/local -xzf ./go1.9.3.linux-amd64.tar.gz              # Install Go.
rm ./go1.9.3.linux-amd64.tar.gz                                       # Delete the installer.
mkdir $GOPATH
mkdir $GOPATH/src 
mkdir $GOPATH/bin 
```

Then Edit the **~/.bashrc** with your preferred editor like **vim**, and append 
```
PATH=$PATH:/usr/local/go/bin
GOPATH=~/environment/go
PATH=$PATH:$GOPATH/bin
export GOPATH
```
in the end of the file.

Source the configuration file:
```
. ~/.bashrc
```
------

Now, use command like to check if you have go in your environment
```
go help
```
And you will see something like this:

```
Go is a tool for managing Go source code.

Usage:

        go command [arguments]

The commands are:

        build       compile packages and dependencies
        clean       remove object files
        doc         show documentation for package or symbol
        env         print Go environment information
        bug         start a bug report
        fix         run go tool fix on packages
        fmt         run gofmt on package sources
        generate    generate Go files by processing source
        get         download and install packages and dependencies
        install     compile and install packages and dependencies
        list        list packages
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         run go tool vet on packages
```
------

After you have go in your env, use following command to install dependency check tool "dep"
```
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
```
or
```
go get -u github.com/golang/dep/cmd/dep
```

------
### Step 5:
We are going to reuse previous workshop content by **Pahud** --> https://github.com/pahud/lambda-gin-refarch
```
cd $GOPATH/src
git clone https://github.com/pahud/lambda-gin-refarch.git
cd lambda-gin-refarch
dep ensure -v
```
Create a S3 Bucket for the deploying the GO application
We use "golang-workshop-bucket-1234" as this example.
**Please change the bucket name in your practice**
**And also check if you are using correct region command setting**
```
aws s3api create-bucket --bucket golang-workshop-bucket-1234 --region us-east-1
###
###If you are using any region not us-east-1, please append LocationConstraint
aws s3api create-bucket --bucket golang-workshop-bucket-1234 --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1
```
Edit *Makefile* and update the following variables
```
S3TMPBUCKET	?= golang-workshop-bucket-1234
STACKNAME	?= lambda-gin-refarch
LAMBDA_REGION ?= us-east-1
```
***S3TMPBUCKET*** - change this to your private S3 bucket and make sure you have read/write access to it. This is an intermediate S3 bucket for AWS SAM CLI to deploy as a staging bucket.
***STACKNAME*** - change this to your favorite cloudformatoin stack name.
***LAMBDA_REGION*** - the region ID you are deploying to
```
make world
```
and you will see the output message:
```

Checking dependencies...
Building...
Packing binary...
  adding: main (deflated 64%)

        SAM CLI now collects telemetry to better understand customer needs.

        You can OPT OUT and disable telemetry collection by setting the
        environment variable SAM_CLI_TELEMETRY=0 in your shell.
        Thanks for your help!

        Learn More: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-telemetry.html


        Deploying with following values
        ===============================
        Stack name                 : lambda-gin-refarch
        Region                     : None
        Confirm changeset          : False
        Deployment s3 bucket       : golang-workshop-bucket-1234
        Capabilities               : ["CAPABILITY_IAM"]
        Parameter overrides        : {}

Initiating deployment
=====================
Uploading to 0950cdbeef24ce21e02bb9713a502de6  4127265 / 4127265.0  (100.00%)
Uploading to 41b245584c0d88957ae07013d98a570a.template  734 / 734.0  (100.00%)
Waiting for changeset to be created..

CloudFormation stack changeset
------------------------------------------------------------------------------------------------
Operation                        LogicalResourceId                ResourceType                   
------------------------------------------------------------------------------------------------
* Modify                         SampleFunction                   AWS::Lambda::Function          
* Modify                         ServerlessRestApi                AWS::ApiGateway::RestApi       
------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:ap-southeast-1:384612698411:changeSet/samcli-deploy1584628047/0f79aaa0-233f-4bf2-8421-4b9258d31029


2020-03-19 14:27:37 - Waiting for stack create/update to complete

CloudFormation events from changeset
-------------------------------------------------------------------------------------------------
ResourceStatus           ResourceType             LogicalResourceId        ResourceStatusReason   
-------------------------------------------------------------------------------------------------
UPDATE_IN_PROGRESS       AWS::Lambda::Function    SampleFunction           -                      
UPDATE_COMPLETE          AWS::Lambda::Function    SampleFunction           -                      
UPDATE_COMPLETE_CLEANU   AWS::CloudFormation::S   lambda-gin-refarch       -                      
P_IN_PROGRESS            tack                                                                     
UPDATE_COMPLETE          AWS::CloudFormation::S   lambda-gin-refarch       -                      
                         tack                                                                     
-------------------------------------------------------------------------------------------------

CloudFormation outputs from deployed stack
-------------------------------------------------------------------------------------------------
Outputs                                                                                         
-------------------------------------------------------------------------------------------------
Key                 DemoGinApi                                                                  
Description         URL for application                                                         
Value               https://aeudzdz4mb.execute-api.ap-southeast-1.amazonaws.com/Prod/ping       
-------------------------------------------------------------------------------------------------

Successfully created/updated stack - lambda-gin-refarch in ap-southeast-1


# print the cloudformation stack outputs
aws --region ap-southeast-1 cloudformation describe-stacks --stack-name "lambda-gin-refarch" --query 'Stacks[0].Outputs'
[
    {
        "Description": "URL for application", 
        "ExportName": "DemoGinApi", 
        "OutputKey": "DemoGinApi", 
        "OutputValue": "https://aeudzdz4mb.execute-api.ap-southeast-1.amazonaws.com/Prod/ping"
    }
]
[OK] Layer version deployed.
```
*Get your API Gateway URL*
You will see the API Gateway URL in the OutputValue above. Try request the URL with cURL or http browser:
```
curl -s  https://xxxxxxxxxxx.execute-api.us-east-1.amazonaws.com/Prod/ping | jq -r                                                  
{
  "message": "pong"
}
```

------

### Step 6:
We are going to build a container to deploy the application to ECS-Fargate.
* Edit a *Dockerfile* with following content:
```
FROM golang:1.13

WORKDIR /go/src/app
COPY . .

RUN go get -u github.com/codegangsta/gin \
  && go get -u github.com/golang/dep/cmd/dep \
  && dep ensure

CMD ["go","run","/go/src/app/main.go"]
```
And run following command to build a docker for this go-gin-web-application
```
docker build -t golang-gin-ecs-fargate .
```

Then, create ECR with awscli:
```
aws ecr create-repository --repository-name golang-gin-ecs-fargate
```
And you will see some output message like:
```
{
    "repository": {
        "registryId": "384612698411", 
        "repositoryName": "golang-gin-ecs-fargate", 
        "repositoryArn": "arn:aws:ecr:ap-southeast-1:384612698411:repository/golang-gin-ecs-fargate", 
        "createdAt": 1583828485.0, 
        "repositoryUri": "384612698411.dkr.ecr.ap-southeast-1.amazonaws.com/golang-gin-ecs-fargate"
    }
}
```
Then you can find your ECR Uri.
Before you doing any docker command, you have to get ECR login with this awscli command
```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 384612698411.dkr.ecr.ap-southeast-1.amazonaws.com/golang-gin-ecs-fargate
```
**Note: If you have error message about awscli, you might need to check if the awscli is up to date. (use pip install --upgrade pip && pip install --upgrade awscli)**

```
docker tag golang-gin-ecs-fargate:latest 384612698411.dkr.ecr.ap-southeast-1.amazonaws.com/golang-gin-ecs-fargate:latest
docker push 384612698411.dkr.ecr.ap-southeast-1.amazonaws.com/golang-gin-ecs-fargate:latest
```
------

### Step 7:
* Use CDK to deploy the Docker Image onto ECS-Task on Fargate
* Download the cdk sub-dir in this repository,
```
cd ~/environment
git clone https://github.com/awshktsa/AWSWorkshop-20200313-golang-serverless-ecs-fargate
cd AWSWorkshop-20200313-golang-serverless-ecs-fargate/cdk
```
and you can see following file
```
- cdk.json
- index.ts
- package.json
- tsconfig.json
```

And you need to edit index.ts, which is to point the deployed docker from your repository
```
// and replace the ECR_NAME with your ecr name like XXXXXXX.ecr.ap-southeast-1.amazonaws.com
image: ecs.ContainerImage.fromEcrRepository(ecr.Repository.fromRepositoryName(this,"ECR_NAME","golang-gin-ecs-fargate")),
```
Before we use CDK deploy, you need to setup the environment variable: ***CDK_DEFAULT_ACCOUNT***, and ***CDK_DEFAULT_REGION***
```
export CDK_DEFAULT_ACCOUNT=12345678901234
export CDK_DEFAULT_REGION=us-east-1
```
Everything is ready, now use following command to deploy the stack.
```
npm inistall
npm run build
cdk synth
cdk deploy
```
***And you might see following result in your terminal:***
```
This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬───────────────────────────────────────────┬────────┬───────────────────────────────────────────┬───────────────────────────────────────────┬───────────┐
│   │ Resource                                  │ Effect │ Action                                    │ Principal                                 │ Condition │
├───┼───────────────────────────────────────────┼────────┼───────────────────────────────────────────┼───────────────────────────────────────────┼───────────┤
│ + │ ${FargateService/TaskDef/ExecutionRole.Ar │ Allow  │ sts:AssumeRole                            │ Service:ecs-tasks.amazonaws.com           │           │
│   │ n}                                        │        │                                           │                                           │           │
├───┼───────────────────────────────────────────┼────────┼───────────────────────────────────────────┼───────────────────────────────────────────┼───────────┤
│ + │ ${FargateService/TaskDef/TaskRole.Arn}    │ Allow  │ sts:AssumeRole                            │ Service:ecs-tasks.amazonaws.com           │           │
├───┼───────────────────────────────────────────┼────────┼───────────────────────────────────────────┼───────────────────────────────────────────┼───────────┤
│ + │ ${FargateService/TaskDef/web/LogGroup.Arn │ Allow  │ logs:CreateLogStream                      │ AWS:${FargateService/TaskDef/ExecutionRol │           │
│   │ }                                         │        │ logs:PutLogEvents                         │ e}                                        │           │
├───┼───────────────────────────────────────────┼────────┼───────────────────────────────────────────┼───────────────────────────────────────────┼───────────┤
│ + │ *                                         │ Allow  │ ecr:GetAuthorizationToken                 │ AWS:${FargateService/TaskDef/ExecutionRol │           │
│   │                                           │        │                                           │ e}                                        │           │
├───┼───────────────────────────────────────────┼────────┼───────────────────────────────────────────┼───────────────────────────────────────────┼───────────┤
│ + │ arn:${AWS::Partition}:ecr:ap-southeast-1: │ Allow  │ ecr:BatchCheckLayerAvailability           │ AWS:${FargateService/TaskDef/ExecutionRol │           │
│   │ 384612698411:repository/golang-gin-ecs-fa │        │ ecr:BatchGetImage                         │ e}                                        │           │
│   │ rgate                                     │        │ ecr:GetDownloadUrlForLayer                │                                           │           │
└───┴───────────────────────────────────────────┴────────┴───────────────────────────────────────────┴───────────────────────────────────────────┴───────────┘
Security Group Changes
┌───┬─────────────────────────────────────────────────┬─────┬────────────┬─────────────────────────────────────────────────┐
│   │ Group                                           │ Dir │ Protocol   │ Peer                                            │
├───┼─────────────────────────────────────────────────┼─────┼────────────┼─────────────────────────────────────────────────┤
│ + │ ${FargateService/LB/SecurityGroup.GroupId}      │ In  │ TCP 80     │ Everyone (IPv4)                                 │
│ + │ ${FargateService/LB/SecurityGroup.GroupId}      │ Out │ TCP 80     │ ${FargateService/Service/SecurityGroup.GroupId} │
├───┼─────────────────────────────────────────────────┼─────┼────────────┼─────────────────────────────────────────────────┤
│ + │ ${FargateService/Service/SecurityGroup.GroupId} │ In  │ TCP 80     │ ${FargateService/LB/SecurityGroup.GroupId}      │
│ + │ ${FargateService/Service/SecurityGroup.GroupId} │ Out │ Everything │ Everyone (IPv4)                                 │
└───┴─────────────────────────────────────────────────┴─────┴────────────┴─────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)? 
```
***press Y <enter> then it will start to deploy the ECS Fargate for you***
        
### After Workshop -- Clean up
* *** clean up the ECS-Fargate stack with "cdk destroy"***
* *** cloud up the Lambda SAM with "aws cloudformation delete-stack --stack-name=lambda-gin-refarch"***
* *** Terminate the Cloud9 Env from AWS Console > Cloud9 > Environment > Delete***
* *** Remove IAM Role/User from AWS Console > IAM > Role/User > Delete***
