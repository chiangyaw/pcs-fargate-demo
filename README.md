# Prisma Cloud Demo for AWS Fargate
This is a guide to demonstrate Prisma Cloud integration and capabilities with AWS Fargate. In this demo, we will be using `vulnerables/web-dvwa` Docker container, deployed as a Fargate in a ECS cluster. We will be running through how Prisma Cloud is able to prevent threats on AWS Fargate. 

### Prerequisties

- Terraform installed
- AWS credentials configured (via `~/.aws/credentials` or environment variables)
- AWS account with appropriate permissions for VPC, ECS, and IAM
- Access to a Prisma Cloud tenant

### Demo Flow

#### Deploying Fargate with App-Embedded Defender
1. Deploy Fargate via the Terraform scrips [here](https://github.com/chiangyaw/aws-dvwa-fargate)

2. Once completed, make sure you're able to access the DVWA page via the public IP address given.

3. To provide protection capability to AWS ECS Fargate, Prisma Cloud will be utilizing a deployment method called App-Embedded Defender for Fargate. For more information, refer to [here](https://docs.prismacloud.io/en/compute-edition/22-12/admin-guide/install/install-defender/install-app-embedded-defender-fargate)

4. To generate the Task Definition, go to Prisma Cloud tenant > Runtime Security > Defender > Manual deploy.

5. Choose Single Defender > Container Defender - App-Embedded. Deployment Type choose Fargate task. We will need the task definition from AWS console here.

6. Go to AWS console > Amazon ECS > Task definitions > dvwa-task > Choose the latest version > Create new revision > Create new revision with JSON. Copy the entire JSON, paste it into Prisma Cloud (previous step) under "Insert task definition". Click Generate protected task and click Copy protected task, and replace the JSON file on AWS console and click Create. By doing this, the DVWA task definition has been updated on AWS side with App-Embedded Defender.

7. On AWS console, click on Deploy > Update Service. Choose the correct Cluster and Service, tick on Force new deployment and click Update. 

8. Make sure the new Deployment is succcessful and running. To cross check this, you can headback to Prisma Cloud, you should see the Defender deployed and connected.

Note: The public IP address to access the Fargate might change. You can recheck the new public IP address in AWS Console > ECS > Clusters > dvwa-cluster > Services > dvwa-service > Task > Choose the latest running Tasks > Networking > Public IP.

#### Demo 1 - SQL Injection with WAAS
1. Connect to the DVWA page, and login with the credential 'admin' and 'password'. If this is the first time you're logging in, you will need to reset the database.

2. On the DVWA page, click on SQL Injection, and paste the following into the text box and click "Submit". You should see some output where it shows SQL Injection is successful.
    ```
    1' or 1=1#
    ```

3. To prevent this, we will need to setup WAAS rule. Go to Prisma Cloud > Runtime Security > DEFEND > WAAS > App-Embedded > Add rule. Give it any name, and on the Scope, create a new collection. Give it any name, and insert the task name in the App ID with wildcard. It should be ```dvwa-task*```. Click Save.

4. Click on the rule, click Add app and Add endpoint in the new pop up window. Type 80 for App ports and 8080 for WAAS port, and click Create.
    Note: To access the web app with WAAS protection, you will need to access via 8080 now. Prisma Cloud WAAS is now sitting inline with the new port, and forward the web traffic to the existing app port. 

5. Move to App firewall tab, change everything as Prevent, and click Save. 

6. Now, access the DVWA page with port 8080, and do the same thing on SQL Injection page. You will see a pop up page stating Access denied by Prisma Cloud. You can also look into Prisma Cloud > Runtime Security > MONITOR > Event to check on the prevent logs.

#### Demo 2 - Command Injection with WAAS
1. Now you have 2 ways to access DVWA page, one with port 80 (unprotected) and another with port 8080 (protected). 

2. On DVWA page (unprotected), click on Command Injection. Type the following and click "Submit". You should see the list of items on the passwd file.
    ```
    127.0.0.1| cat /etc/passwd
    ```

3. Do the same on the protected DVWA page, check on the result and logs.


#### Demo 3 - Runtime Protection
1. For runtime, you will need to create a runtime rule. Go to Prisma Cloud > Runtime Security > DEFEND > Runtime > App-Embedded policy > Add rule.

Work in Progress