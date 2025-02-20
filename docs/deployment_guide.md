# Deployment Guide
## Requirements
Before you deploy, you must have the following in place:
*  [AWS Account](https://aws.amazon.com/account/) 
*  [GitHub Account](https://github.com/) 
*  [AWS CLI](https://aws.amazon.com/cli/) 
*  [AWS SAM](https://aws.amazon.com/serverless/sam/) 


## Step 1: Setup the Auth Provider (Google)

If your don't have a Google OAuth client and secret key, please, follow these instructions:

1.	Go to [Google developer console](https://console.developers.google.com)
2.  Click select a project
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google1.png)
3.  Click SELECT A PROJECT and then NEW PROJECT
   ![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google2.png)
4.  Type in project name and click CREATE
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google3.png)
5.  Once the project is created, from the left Navigation menu, select APIs & Services, then select Credentials
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google4.png)
6.  Click CONFIGURE CONSENT SCREEN
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google5.png)
7.  Click CREATE
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google6.png)
8.  Type in App Information and Developer contact information which are required field and click SAVE AND CONTINUE three times (OAuth consent screen -> Scopes -> Test Users) to finish setting up consent screen.
9. Back to Credentials tab, Create your OAuth2.0 credentials by choosing OAuth client ID from the Create credentials drop-down list.
	![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google7.png)
10. Choose Web application as Application type and name your OAuth Client.
11. Click Create.
12. Take note of Your **client ID** and Your **Client Secret**. You will need them for the next section.
13. Choose OK.

 
## Step 2: Front-end deployment

The **Deploy to Amplify Console** button will take you to your AWS console to deploy the front-end solution.

<a href="https://console.aws.amazon.com/amplify/home#/deploy?repo=https://github.com/arturlr/minecraft-server-dashboard">
    <img src="https://oneclick.amplifyapp.com/button.svg" alt="Deploy to Amplify Console">
</a>


The follow screenshots shows how simple this step is:

1. When you click at the button above, please make sure AWS Amplify hosting opens into the context of your AWS Account Console in your browser. You will be presented with the page below asking to connect to your Github account. Amplify will fork the Minecraft dashboard to deploy it.
   
   ![alt text](../images/amplify-console-01.png)

2. To deploy the solution you need to select an Amplify Role. click at **Create new role** button. It will open a new tab in your browser with a pre-defined role configuration for an Amplify IAM Role. Accept all the default options until the IAM role is created and move to the next step. 
   
   ![alt text](../images/amplify-console-02.png)

3. The Deploy App configuration should now have the IAM Role as the picture below. If not click on the refresh icon and select the IAM Role. Next step is to configure the **Enviroment variable** with the *Google Client ID* and *Google Client Secret* that will be used for authentication. This is how Amplify passes this configuration along to Amazon Cognito. **USE THE VALUES YOU OBTAINED IN STEP 1**
   
   ![alt text](../images/amplify-console-03.png)
   ![alt text](../images/amplify-console-04.png)


4. After you have all the configuration done for the deployment, Amplify will fork the Github repo, create the app in Amplify and kicks-off the application deployment.
   
   ![alt text](../images/amplify-console-06.png)

5. The deployment will take a few minutes. The green icons of the CI/CD deployment process indicates that the solution deploy successfuly. The Dashboard URL below of the image of a window with the Amazon icon on it. It wont work yet as we still have some configuraton to do.
   ![alt text](../images/amplify-console-07.png)


## Step 3: Back-end deployment

Make sure you execute the commands below using bash under an IAM user who has permission on your AWS Account. More information at: [AWS CLI Configuration basics](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)

In this step we create the various AWS Lambda Functions that interact with AppSync and the fetches the information from the EC2 Server, the AWS Step-Functions that coordinates the instance start/stop, the AWS Systems Manager Parameter that holds the Amazon Cloudwatch agent configuration, the Amazon DynamoDb used to store the instance alarm's configuration and Minecraft initialization commands and the IAM Role that is associated with the instance at the instance start.

The backend relies on resources that needed to be created during the front-end deployment, please make sure to proceed after Amplify has successfully deployed the application.

The first step is to clone the repo. 
```bash
git clone git@github.com:arturlr/minecraft-server-dashboard.git 
```

The AWS Lambdas have some libraries dependencies. The following commands instructs SAM to build all the dependencies:

```bash
sam build
```

After executing it, you will know that it succeeded when finishing with the message below:

```bash
Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml
```

To deploy the solution you execute the command below. It will guide you 

```bash
sam deploy --guided
```

The output should be similar to the example below. **YOU MUST PROVIDE A GMAIL ACCOUNT AT THE PARAMETER ADMIN EMAIL** that will be used to log as the admin, otherwise you won't be able to do anything at the dashboard.

```bash
Configuring SAM deploy
======================

	Looking for config file [samconfig.toml] :  Not found

	Setting default arguments for 'sam deploy'
	=========================================
	Stack Name [sam-app]: minecraft-dasboard-backend
	AWS Region [us-west-2]: THE_REGION_OF_DEPLOYMENT
	Parameter ProjectName [minecraft-dashboard]:
	Parameter AppValue [minecraft]:
	Parameter AdminEmail []: YOUR_GMAIL_ACCOUNT@gmail.com
	#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
	Confirm changes before deploy [y/N]: y
	#SAM needs permission to be able to create roles to connect to the resources in your template
	Allow SAM CLI IAM role creation [Y/n]: y
	#Preserves the state of previously provisioned resources when an operation fails
	Disable rollback [y/N]: y
	Save arguments to configuration file [Y/n]: y
	SAM configuration file [samconfig.toml]:
	SAM configuration environment [default]:
```

Confirm the deploy of the changeset and wait for the to finish. At the end you should see a message like:

```
Successfully created/updated stack - minecraft-dasboard-backend in AWS_REGION_OF_DEPLOYMENT
```

## Step 4: Allowing Amazon Cognito authenticate into Google Federation

1. Go to the AWS Console and select the Cognito User Pool at the region the applicatoin was deployed and select the one that starts with **minecraftdasboard**.
2. Select the tab **App Integration** and copy the cognito domain URL.
   ![alt text](../images/cognito-domain.png)
3. Go to [Google developer console](https://console.developers.google.com)

4. On the left navigation bar, choose Credentials.

5. Select the client you created in the first step and click the edit button.

6. Paste your user pool domain into **Authorized Javascript origins**.

7. Paste your user pool domain with the /oauth2/idpresponse endpoint into **Authorized Redirect URIs**.
   ![alt text](https://docs.amplify.aws/images/cognitoHostedUI/google8.png)


