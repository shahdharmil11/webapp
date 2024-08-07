# Cloud Native Web Application

## Prerequisites

Before we begin, we need to fulfill the following requirements:

- Java Development Kit (JDK) 17
- Maven --> pom.xml with relevant dependancies
- PostgreSQL Database --> credentials that are stored

## Build Instructions

To build the application, follow these steps:

Open terminal:
  1. Clone the repository on relevant path: git clone https://github.com/your-username/your-repository.git
  2. Open the project, check the github actions workflow and branch rules
  3. Make sure port 5432 is available for DB connection
  4. Make sure port 8080 is available for tomcat connection
  5. Complete Maven lifecycle to resolve dependency issues
  6. Run Test cases and create few instances
  7. Make sure CI/CD is working as expected

<img src="https://github.com/shahdharmil11/webapp/blob/master/gcp-infra.jpeg"  title="Infra provisioned">

## Repositories  

- [**WebApp**](https://github.com/shahdharmil11/webapp): Spring Boot-based RESTful service for user management with PostgreSQL.

- [**Terraform Infrastructure (tf-gcp-infra)**](https://github.com/shahdharmil11/terraform-infra-gcp): Terraform code to provision and manage the GCP infrastructure.

[//]: # (## Getting Started)

[//]: # (To utilize this suite, follow the setup instructions in the README of each repository.)

## Use Cases
- Automated cloud deployments
- Accelerated autoscaling and consistent infra with Packer golden images  
- Event-driven serverless workflows
- Infrastructure as code for GCP


### Assignment 1
- Start Spring Boot application  

- GET request with no payload, no query params
```
curl -vvvv http://localhost:8081/healthz
```  
[expected: 200 status ok]  

- GET request with query params
```
curl -vvvv http://localhost:8081/healthz?key=value
```  
[expected: 400 bad request]  

- GET request with payload
```
curl -vvvv -X GET http://localhost:8081/healthz --data-binary '{"key":"value"}' -H "Content-Type: application/json"
```  
[expected: 400 bad request]

- PUT/POST/PATCH/DELETE request
```
curl -vvvv -XPUT http://localhost:8081/healthz
```  
[expected: 405 method not allowed]  

- Stop Postgresql, make GET request
```
net stop postgresql-x64-16
curl -vvvv http://localhost:8081/healthz
```
[expected: 503 service unavailable]  

- Start Postgresql, make GET request
```
net start postgresql-x64-16
curl -vvvv http://localhost:8081/healthz
```
[expected: 200 status ok]

Notes:
- Decreased the time it takes to recognize a non-operational database to 5 secs via HikariCP  

### Assignment 2  

- Bootstrapping the Database
  
-- Schema Management: Our use of JPA entity classes, in combination with Spring Data JPA repositories, facilitates the auto-generation of database tables. This negates the need for manual SQL query execution to restore tables should they be dropped.  

- REST APIs  

-- POST request to create user  

Endpoint:
```
http://localhost:8081/v1/user
```
Request Body:
```
{
  "first_name": "Jane",
  "last_name": "Doe",
  "password": "asqq@1A",
  "username": "jane.doe@example.com"
}
```
Responses:
```
- 201 created
{
  "id": "d290f1ee-6c54-4b01-90e6-d701748f0851",
  "first_name": "Jane",
  "last_name": "Doe",
  "username": "jane.doe@example.com",
  "account_created": "2016-08-29T09:12:33.001Z",
  "account_updated": "2016-08-29T09:12:33.001Z"
}
- 400 bad request 
```
-- GET request to fetch user details  
(basic authentication token needed to make API call) 

Endpoint:
```
http://localhost:8081/v1/user/self
```  
Responses:
```
- 200 status ok
{
  "id": "d290f1ee-6c54-4b01-90e6-d701748f0851",
  "first_name": "Jane",
  "last_name": "Doe",
  "username": "jane.doe@example.com",
  "account_created": "2016-08-29T09:12:33.001Z",
  "account_updated": "2016-08-29T09:12:33.001Z"
}
- 401 unauthorized
```
-- PUT request to update user details  
  (basic authentication token needed to make API call)

Endpoint:
```
http://localhost:8081/v1/user/self
{
  "first_name": "Jane",
  "last_name": "Doe",
  "password": "skdjfhskdfjhg"
}
```  
Responses:
```
- 204 no content
- 400 bad request
- 401 unauthorized
```

- GitHub actions CI Pipeline  

-- Wrote a GitHub Actions workflow in yml to run simple check (compile code) for each pull request raised.   
-- A pull request can only be merged if the workflow executes successfully.  


- Bash script to demo assignment-2 in a centos vm  

-- The script installs necessary packages like unzip, JDK, maven, postgresql in the vm      

-- Unzips the project folder, creates directory 'resources' under webapp/src/main, and places application.properties file inside it    

-- Initializes and starts PostgreSQL  

-- Configures PostgreSQL  

-- Configures md5 authentication in pg_hba.conf  

-- Makes maven point to JDK 17 instead of the default mapping to JDK 1.8 

-- Builds and runs the Spring Boot application  

### Assignment 3

- Added Integration tests  

-- Test 1 - Created an account, and using the GET call, validated account exists  
-- Test 2 - Updated the account and using the GET call, validated the account was updated 


- Added GitHub secrets to encrypt sensitive information in CI workflow  

-- Created secrets in organization repo  
-- Enabled 'Run workflows from fork pull requests' and 'Send secrets and variables to workflows from fork pull requests' in order to give access of secrets to fork repo  
-- Accessed the secrets in workflow file through ${{ secrets.VARIABLE_NAME }}  

- Handled via Terraform (repo: 'tf-gcp-infra')  

-- Provisioned VPC, subnets (public subnet for 'webapp', private subnet for 'database')  
-- Provisioned route explicitly to 0.0.0.0/0 with next hop to Internet Gateway and attached to VPC  


### Assignment 4  

- Built custom Packer image via GitHub Actions workflow that will get triggered when PR gets merged into organization repo's main branch    


- Packer image had the following steps:  

-- Shell provisioner to update OS, install Postgres database locally inside Packer image, install dependencies   
-- File provisioner to copy build file (.jar) generated by GitHub Actions workflow to Packer image  
-- Shell provisioner to move jar file to new directory /opt/webapp, create no-login user (csye6225) and group, make this user the owner of all the web application related artifacts  
-- File provisioner to copy systemd service file (to turn our web application into a service that can be started/stopped via systemctl commands) to Packer image  
-- Shell provisioner to move systemd service file to /etc/systemd/system/webapp.service, and start the service


- Created Service Account in GCP for GitHub actions to deploy Packer image [added this service account credentials as GitHub organization repository secret, and used them in the github actions workflow so that Github actions can use it to deploy packer image to GCP]


- Added new GitHub Actions workflow for status check (gets triggered on raising a PR from fork/feature to org/main)  

-- If packer fmt fails, workflow fails  
-- If packer validate fails, workflow fails  


- Handled via Terraform (repo: 'tf-gcp-infra')  

-- Provisioned Google Compute Engine instance (VM) from custom Packer image, with VM startup script that just restarts the web application service file  
-- Set up firewall rules for VPC/Subnet to allow traffic from the internet to the application port [8081]

### Assignment 5  

- Updated Packer template with following steps:  

-- Shell provisioner to create empty application.properties file, make no-login user 'csye6225' the owner (this empty file will be populated with cloud DB details via VM startup script once provisioned via Terraform)  
-- Removed shell provisioner to install Postgres DB locally inside Packer image (since local DB will be replaced with Google CloudSQL provisioned via Terraform)


- Handled via Terraform (repo: 'tf-gcp-infra')  

-- Setup Private Services access in VPC to access Google CloudSQL instance  
-- Provisioned CloudSQL instance (Postgres 14), CloudSQL Database, CloudSQL Database User  
-- Provisioned Google Compute Engine instance (VM) from custom Packer image, with VM startup script that passes the database configuration such as username, password, and hostname to the web application and starts it  
-- Added Network security (CloudSQL instance should not be accessible from the internet, can only be accessed by the compute engine instance running the web application.)  

### Assignment 6  

- Registered domain [gcp-infra-dharmil.me] with Namecheap  


- Create a public zone in Cloud DNS manually from the GCP console for gcp-infra-dharmil.me  


- Configure Namecheap to use custom name servers provided by GCP  


- Verified name server setup with command: 

```
dig NS gcp-infra-dharmil.me
```  

- Handled via Terraform (repo: 'tf-gcp-infra')  

-- Added/updated A record to the Cloud DNS zone so that domain [gcp-infra-dharmil.me] points to public IP of VM instance  
-- Created service account for VM with IAM roles 'Logging Admin' and 'Monitoring Metric Writer' for the Ops agent (to be installed) to collect and send logs to Google Log Explorer  


- Updated Packer template with following steps:  

-- Shell provisioner to create log directory and set ownership so that application can write logs to it  
-- Shell provisioner to install Ops Agent for collecting VM logs and sending them to GCP Cloud Logging  
-- File provisioner to copy Ops agent configuration file to Packer image  
-- Shell provisioner to move Ops config file to /etc//google-cloud-ops-agent/config.yaml, enable and start google-cloud-ops-agent.service  

- Web Application Updates  

-- Updated web application to write Structured Logs in JSON (used SLF4J with Logback)  
-- Created logback config file to use this directory '/var/log/webapp/webapp.log' to write logs  
-- All application log data is streamed to Google Cloud Observability and available in Log Explorer.  

### Assignment 7  

Objective: When POST call is made to the web application to create a new user, the web application will publish a message to a Pub/Sub topic with the email id of the user. The Pub/Sub will trigger an associated CloudFunction which will use Mailgun's API to send an email verification link to the user. This link if clicked by the user within 2 minutes, the user's email will be verified, else not. All the subsequent GET/PUT calls will check if the user is a verified user or not. If yes, only then they will be able to make the API calls. 

- Handled via Terraform (repo: 'tf-gcp-infra')  

-- Provisioned Cloud Function, Pub/Sub, VPC Connector (for cloud function to interact with the Cloud SQL Postgres db), service account for the Cloud Function with necessary IAM role bindings  

- Web Application Updates 

-- Publishes a message to Pub/Sub topic when a new user account is created. The payload (message) is in JSON  
-- Once the message is published to Pub/Sub and Cloud function is triggered and verification link is sent to user, and user clicks that link, a new REST API endpoint in the web application ('/verify') is hit which checks if the link is clicked by user within 2 minutes. If yes, then marks the user as 'verified'  
-- All API calls from user account that has not been verified are blocked until the user completes the verification  

- Serverless  

-- Created new GitHub repo to store the serverless code of Cloud Function (used Java)  
-- The Cloud Function will do the following:  

1. Receive the base64 encoded message (containing email id of the user) from the Pub/Sub topic  
2. Decode the message, deserialize it, insert a verification token in the cloudsql db associated with the email id of the user, and also add a timer of 2 minutes  
3. Send mail to the user using mailgun's api that contains the verification url. When user clicks the link, it redirects them to the '/verify' endpoint of the web application   

### Assignment 8  

- Setup Autoscaling and Application Load balancer for the web application  

### Assignment 9  

Updated the Packer build workflow to include these steps:  

- Created a new Instance Template version with the latest machine image id for the managed instance group.  
- Configured the managed instance group to use this new template using gcloud cli.  
- Issued command to the managed instance group to start a basic rolling update using gcloud cli.  
- GitHub Actions workflow waits for managed instance group refresh to complete before exiting. The status of GitHub Actions workflow matches the status of instance refresh command.
