# Create a Linux EC2 instance and run these commamnds
sudo yum install git -y
sudo yum install docker -y
sudo yum install wget -y
sudo wget https://github.com/awanmbandi/http-web-application/raw/zips/http-web-application-master.zip
sudo unzip http-web-application-master.zip
	Create an ECR Repository
Navigate to ECR
create repo
Visibility: Private
name it <http-web-application>
Enable scan on push
Disable Encryption
Create repo
	Create a code commit
create repo
Name it <http-web-application>
describe itCreate
Clone URL
Clone HTTPS
Terminal
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/http-web-application
Enter Username: HTTPS Git credentials for AWS CodeCommit in IAM
Password: HTTPS Git credentials for AWS CodeCommit in IAM
ls
cd http-web-application
ls
# Copy the content of http-web-application-master into the new repository http-web-application
sudo cp -rf ../http-web-application-master/* .
ls
cat Dockerfile
# Build docker image using instructions in dockerfile
sudo docker build -t <ecr repo URI> version present working directory
sudo docker build -t 765878755895.dkr.ecr.us-east-1.amazonaws.com/http-web-application:1.0 .
sudo docker images
sudo sudo
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 765878755895.dkr.ecr.us-east-1.amazonaws.com
docker images
	*** Push image to the CodeCommit repository ***
docker push 765878755895.dkr.ecr.us-east-1.amazonaws.com/http-web-application:1.0
ls
git status
git add .
git commit -m "adding web application source code"
git push
Enter Username: HTTPS Git credentials for AWS CodeCommit in IAM
Password: HTTPS Git credentials for AWS CodeCommit in IAM
	*** Add a build.spec file for the build job ***
# Download the build spec file
sudo wget https://raw.githubusercontent.com/awanmbandi/http-web-application/zips/buildspec.yml
ls
ls -al
vi buildspec.yml
edit the buildspec.yml
	Pre-build: repository URI: 765878755895.dkr.ecr.us-east-1.amazonaws.com/http-web-application
	Post-build printf: container name is ECS/Task definition, rev 1: http-web-application-cd
:wq!
git status
git add .
git commit -m "adding web application source code"
git push
Enter Username: HTTPS Git credentials for AWS CodeCommit in IAM
Password: HTTPS Git credentials for AWS CodeCommit in IAM
	*** Create a task definition ***
Navigate to ECS
Task definitions
Create new task definition
EC2, Next
Name: http-web-application
Task Role: select 1
Network mode: definition
Task execution role: select same role sected at the top
Add container
	Container name: http-web-application-cd
	Image: ecr http-web-application 1.0 URI: 765878755895.dkr.ecr.us-east-1.amazonaws.com/http-web-application:1.0
	Memory Limits: 128
	Host port: 0 (dynamic port mapping)
	Container port: 8000
	Everything else: default
	Add
Create
	*** Deployment of the application ***
	Create a service
Navigate to ECS Clusters
Services
create service
EC2
Family: http-web-application
service name: http-web-application-service
Number of tasks: 3
Next
LB Type: Application LB
Service IAM role: cREATE NEW
Load balancer name: default
CREATE LB
	Production listener port: 80:HTTP
	Target group name: target
Next, Next, Create
	*** Edit Security group of EC2 instance created in ECS ***
Navigate to ECS/Clusters/ecs-project-ec2/ECS Instances
Open the Instance
Security
Open Security group
Inbound rules: Edit (should be bland or delete the existing)
Add rule
	All Taffic, everywhere, save rules
	
Navigate to EC2, LB, copy LB DNS and paste in a browser.
Refresh and if getting error message: create revision 2
Navigate to ECS
Task definition
 http-web-application
 http-web-application:1
create new revision
Add container: open http-web-application-cd
Image: awanmbandi/http-web-app:1.0
update
create
Navigate to CodeCommit
CodeBuild
	*** BUILD ***
Project name: http-web-app-build-job
description: http-web-app-build-job
Source provider: AWS CodeCommit
Repository: http-web-application
Branch: master
Environment image: Managed image
OS: Linux 2
Runtimes: Standard
Image 4.0
Image version: Always use latest image for runtime
Env type: Linux
Privileged: selected
Service role: New service role
Role name: codebuild-http-web-app-build-job-service-role
Build specifications: Use a buildspec file
Buildspec name: buildspec.yml
Logs:
	Enable Cloudwatch
	Group name: http-web-app-build-job
	Stream name: http-web-app-build-job
Create build project
	*** PIPELINE ***
Create pipeline
Pipeline name: http-web-app-pipeline
Next
Source provider: AWS CodeCommit
Repository name:
Branch name:
Change detection options:
Next
Build provider: codebuild
Project name: http-web-app-build-job
Next
Deploy provider: ECS
Cluster name: ecs-project-ec2
Service name: http-web-application-service
Image definitions file: imagedefinitions.json
Next
Create
The build will break at this point
Navigate to IAM
Roles
Search: codebuild-http-web-app-build-job-service-role
	Add administratorAccess permission
Navigate back to pipeline
Release change