This repository serves as an example to enable tracing for an App Runner service using AWS Copilot CLI. This README is only a short summary of how to deploy the application. For more, please read the [blog post here](https://aws.amazon.com/blogs/containers/enabling-aws-x-ray-tracing-for-aws-app-runner-service-using-aws-copilot-cli/)!

### 1. Deploy your application
```bash
$ copilot init --app movies-app --name api-svc --type "Request-Driven Web Service" --deploy
```
This command will pick up the configuration specified in `copilot/api-svc/manifest.yml` and deploy the service and its supporing resources. In addition, it will also pick up `copilot/app-svc/addons/moviesTable.yml` and deploy the DynamoDB specified by the file along with the service.

You will see the progress as Copilot deploys the service:
```bash
Manifest file for service api-svc already exists. Skipping configuration.
Ok great, we'll set up a Request-Driven Web Service named api-svc in application movies-app.

✔ Created the infrastructure to manage services and jobs under application movies-app.

✔ The directory copilot will hold service manifests for application movies-app.

✔ Manifest file for service api already exists at copilot/api-svc/manifest.yml, skipping writing it.
Your manifest contains configurations like your container size and port.

✔ Created ECR repositories for service api-svc.

✔ Linked account <Your AWS account ID> and region <Your AWS region> to application movies-app.

✔ Proposing infrastructure changes for the movies-app-test environment.
- Creating the infrastructure for the movies-app-test environment.            [create complete]  [90.5s]
  - An IAM Role for AWS CloudFormation to manage resources                    [create complete]  [16.4s]

✔ Created environment test in region us-east-1 under application movies-app.
Building your container image: docker build -t ...

# ~ snip ~

✔ Proposing infrastructure changes for stack movies-app-test-api-svc
- Creating the infrastructure for stack movies-app-test-api-svc                   [create complete]  [409.5s]
  - An IAM Role for App Runner to use on your behalf to pull your image from ECR  [create complete]  [14.6s]
  - An Addons CloudFormation Stack for your additional AWS resources              [create complete]  [67.8s]
    - An IAM ManagedPolicy for your service to access the moviesTable db          [create complete]  [12.5s]
    - An Amazon DynamoDB table for moviesTable                                    [create complete]  [33.4s]
  - An IAM role to control permissions for the containers in your service         [create complete]  [16.7s]
  - An App Runner service to run and manage your containers                       [create complete]  [311.7s]
✔ Deployed service api-svc.
Recommended follow-up action:
 - You can access your service at https://<Random string>.<Your AWS region>.awsapprunner.com over the internet.
```

You should be able to access the URL ⬆️ after the deployment. The page should look like this:
<img width="1774" alt="Screen Shot 2022-03-17 at 2 23 02 PM" src="https://user-images.githubusercontent.com/79273084/167515601-4a7978e4-977f-4532-9a6c-c1c2f8f2ec17.png">


### 2. Populate your DynamoDB
#### Find your table name
 ```shell
$ copilot svc show --app movies-app --name api-svc

About

 Application movies-app
 Name api
 Type Request-Driven Web Service

Configurations

 Environment CPU (vCPU) Memory (MiB) Port
 ----------- ---------- ------------ ----
 test        1          2048         8080

Routes

 Environment URL
 ----------- ---
 test        https://<Random string>.<Your AWS region>.awsapprunner.com

Variables

 Name                     Environment Value
 ----                     ----------- -----
 COPILOT_APPLICATION_NAME test        movies-app
 COPILOT_ENVIRONMENT_NAME "           test
 COPILOT_SERVICE_NAME     "           api
 MOVIES_TABLE_NAME        "           movies-app-test-api-moviesTable # <-- Note this down.
 ```
Note down the value of the environment variable `MOVIES_TABLE_NAME`. It should be a value that’s the same as or similar to `movies-app-test-api-moviesTable`.

### Run the script to populate your DynamoDB

Visit [AWS CloudShell Console](https://ap-northeast-1.console.aws.amazon.com/cloudshell/home?region=ap-northeast-1#) (The link is for "ap-northeast-1". You will need to switch to the region that you deployed your service too. It is normally the region of your default AWS profile).

From AWS CloudShell, run
```
$ git clone git clone https://github.com/Lou1415926/movies-app
$ MOVIES_TABLE_NAME=<The value you note down> bash load-data-script/LoadData.sh 
```
This will populate your DynamoDB table.

After the process is finished, you can visit the URL you were given to access the service and query the database. For example, suppose the URL is https://my-service.us-east-1.aws.dev, you should be able to view the movie information of “Amelia” at https://my-service.us-east-1.aws.dev/api/movie?year=2009&title=Amelia.

