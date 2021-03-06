# sip-stack

Following are the steps to setting up the sip stack 

## Setup SIP middleware repository 
Use this [Cloudformation template](/cloudformation/SIP-stack-repo-publisher-step-1.yaml) to create SIP middleware app repo. 

Following are prerequisites for creating a stack. 
 1. Admin Cloud formation role 
 2. Select IAM resource compatibility check box while running stack

There are three outputs of the stack. 


1. Credentials to push the Middleware repository 
   * *AccesskeyID* 
   * *SecretKey*
2. Repository path as variable *RepoForSipMiddlewareApp*

Download the sip-stack repo and check the [constants](constants.js) based on the region you are creating the stack. Build the Docker using docker build.  If you are using the *us-west-2* zone you can continue with next step without any changes.  

``` docker build -t sip-middleware-app . ```

Following are commands to push docker image to the repository from your computer. 

1. Login to AWS using the credentials generated by the stack. 

```aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin {{REPO - PATH}} ```

2. Tag the docker image 

``` docker tag sip-middleware-app:latest {{REPO - PATH}}:{{REPO- NAME}}:latest ```

3. Push the image to AWS repository and copy the image link  *{{REPO - PATH}}:{{REPO- NAME}}:latest*.
``` docker push *{{REPO - PATH}}:{{REPO- NAME}}:latest* ```

## SIP Stack with sip server and middleware

For creating the SIP stack along with middleware and SIP server using [cloudformation template](cloudformation/Sip-stack-vpc-publisher-step-2.yaml).

Following are the prerequisite of creating before creating this stack
-  Cloudformation stack role with admin rights
- Select IAM resource compatibility check box while running stack
- Region should be same as the first stack 
- AWS EC2 Key exits for launching EC2 instances. 
- SIP Middleware App's image path which we generated in step one. 

Output of this stack is one SIP server EIP which has following ports open 
>  5060 - TCP Allowed to connections from internet.
>  6060 - UDP Allowed to be connected from internet
>  9022 - TCP Allowed to connect from VPC CIDR. 

The middleware connects to the SIP server using internal DNS hosted zone. There is no autocaling configured for this project because for larger system you will have to consider synchronization and a SIP Proxy which is notified by the DNS changes to accept connection on autoscaling. 
