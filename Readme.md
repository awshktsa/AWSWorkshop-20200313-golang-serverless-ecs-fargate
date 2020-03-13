# AWSTPEWorkshop-20200313-golang-serverless-ecs-fargate
For this workshop, we will have a quick review about how to deploy lambda and ecs-fargate. And this time, we will use "GO" as our target application language.
- Start from 2018, AWS Lambda support Go for developing serverless application - [https://aws.amazon.com/about-aws/whats-new/2018/01/aws-lambda-supports-go/](https://aws.amazon.com/about-aws/whats-new/2018/01/aws-lambda-supports-go/)
And we work with Golang Community to have this workshop together, we will use lambda to build a "Go Hello World - Serverless", and also we will extend the infrastructure to Elastic Container Service - Fargate with "Web service Framework -Gin".
------

In this workshop, you will learn following tools to work with AWS:
1. awscli
2. sam for lambda 
3. cdk

------
### Step 1:
* Switch Region on the AWS console, a drag down menu near right-up corner.
Pick one region close to you.

------

### Step 2: Launch a Cloud9 IDE for our workshop env (Note: if you already had a working environment in your local laptop, then please skip this step and jump to Step 3-b)
* **AWS Console > Services > Cloud9 > Create Environment**
- Enter "Name" and "Description" for your Environment, and click Next Step
- Select "Create a new instance for environment (EC2)" for Environment Type
- Select "t2.micro" for Instance Type
- Select "Ubuntu Server xx.xx" for Platform, and click Next Step
- Input Tag Key and Value if you want, and click "Create Environment"

------
### Step 3: Setup IAM Role/User for this workshop
* 3-a: For user who want to use Cloud9
- ** AWS Console > Services > IAM > Role **
- Create Role, service pick "EC2" and click "Next"
- Search "AmazonS3FullAccess" and click the check box
- Search "AWSLambdaFullAccess" and click the check box
- Search "AmazonEC2ContainerRegistryFullAccess" and click the check box
- Search "AmazonECS_FullAccess" and clieck the check box
- Click Next, Inut Tag Key and Value if you want, click Next to enter "Name" and "Description" for the Role.
- Input "golang-workshop-cloud9-role" for the "Name" and click "Create".

* 3-b: For user who want to use your own laptop instead of using Cloud9
- ** AWS Console > Services > IAM > User **
- Create User and attach following policy to this user: ["AmazonS3FullAccess","AWSLambdaFullAccess","AmazonEC2ContainerRegistryFullAccess","AmazonECS_FullAccess"]
- Generate the credential "ACCESS_KEY" and "SECRET_KEY" and keep it safe, never share with anybody else and remeber to deactivate this user after workshop.
------

### Step 4: Setup Go Development Environment in your Cloud9
- After the IDE launch, click to "bash" tab in the botton of the IDE page
- We will follow the AWS Cloud9 User Guide to setup our Go env - [https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-go.html](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-go.html) has all the detail, and we copy the mendatory shell command in below:

```
sudo yum -y update
wget https://storage.googleapis.com/golang/go1.9.3.linux-amd64.tar.gz # Download the Go installer.
sudo tar -C /usr/local -xzf ./go1.9.3.linux-amd64.tar.gz              # Install Go.
rm ./go1.9.3.linux-amd64.tar.gz                                       # Delete the installer.
```

Then Edit the **~/.bashrc** with your preferred editor like **vim**, and append 
```
GOPATH=~/environment/go
PATH=$PATH:$GOPATH/bin

export GOPATH
```
in the end of the file.

Source the configuration file:
```
. ~/.bashrc
```

Now, use command like to check if you have go in your environment
```
go help
```
And you will see something like this:
**
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
**

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
$ cd $GOPATH/src
$ git clone https://github.com/pahud/lambda-gin-refarch.git
$ cd lambda-gin-refarch
$ dep ensure -v
```
Edit *Makefile* and update the following variables
```
S3TMPBUCKET	?= XXXX-golang-workshop-20200313
STACKNAME	?= lambda-gin-refarch
LAMBDA_REGION ?= us-east-1
```

- ***S3TMPBUCKET*** - change this to your private S3 bucket and make sure you have read/write access to it. This is an intermediate S3 bucket for AWS SAM CLI to deploy as a staging bucket.
- ***STACKNAME*** - change this to your favorite cloudformatoin stack name.
- ***LAMBDA_REGION*** - the region ID you are deploying to
When you complete, Just run make world and you will see the go build, zip the compiled binary main into main.zip and sam deploy to deploy your main.zip bundle AWS Lambda and provision API Gateway together.

```
$ make dep build pack
# now go to cdk directory to deploy the stack

$ cd cdk

$ cdk deploy
...
Do you wish to deploy these changes (y/n)? y
...
Outputs:
CdkStack.APIEndpointXXXXXXXXX = https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
...
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
* Download the cdk sub-dir in this repository, and you can see following file
```
- cdk.json
- index.ts
- package.json
- tsconfig.json
```

And you need to edit index.ts
```
// and replace the ECR_NAME with your ecr name like XXXXXXX.ecr.ap-southeast-1.amazonaws.com
image: ecs.ContainerImage.fromEcrRepository(ecr.Repository.fromRepositoryName(this,"ECR_NAME","golang-gin-ecs-fargate")),
```
Then proceed with following command:
```
npm inistall
npm run build
cdk synth
cdk deploy
```

### After Workshop -- Clean up
* clean up the stack with "cdk destroy"
