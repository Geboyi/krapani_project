## Architecture:
Two architectural options have been designed for the facebook and reddit like application to be hosted in the cloud and be accessible on both the web and mobile. The options are listed below:
+ Hybrid Three Tier Architecture with Serverless and CI/CD.
+ Serverless Architecture with CI/CD.

## __Hybrid Three Tier Architecture with Serveless and CI/CD__
This architecture combines a traditional three-tier structure (web, app, database) with serverless components for flexibility and scalability.
#### Architecture Overview:
##### Domains:
This architecture has two domains. The public and private.
* Public Domain:
Route 53 (DNS routing), CloudFront (CDN for content delivery), IAM (Identity and Access Management), AWS WAF (Web Application Firewall), AWS Shield (Distributed Denial-of-Service protection), AWS Cognito (User authentication), API Gateway (API management for mobile requests), Lambda (Serverless functions for routing and logic), S3 (Static content storage), DynamoDB (NoSQL database).
* Private Domain:
PrivateLink (Secure endpoint access within VPC), VPC (Virtual Private Cloud), ALB (Application Load Balancer) - exterior for web traffic, interior for internal routing, RDS (Relational Database Service), RDS Proxy (Connection pooling and management), ElastiCache Redis (In-memory cache).

##### Flow Pattern:
A user accesses the application through a URL or mobile app. Route 53 directs the request to CloudFront. CloudFront caches static content and forwards requests per request type. Mobile and web requests are forwarded to API Gateway and the exterior ALB respectiviely. The API Gateway handles mobile requests and invokes Lambda_a for routing and authentication. Lambda_a calls Lambda_b for main logic processing.
On the other hand, the exterior ALB distributes web traffic to web tier for static content, to interior ALB for dynamic content and to Lambda_b via privatelink for serverless logic. The interior ALB routes internal traffic to the app tier. Lambda_b interacts with the database tier through RDS Proxy for data operations. RDS Proxy manages database connections and optimizes performance. Processed data is returned through the reverse path to the user.

##### Networking:
- Public Subnets: Web tier and exterior ALB, accessible from the internet.
- Private Subnets: Interior ALB, app tier, database tier, protected from direct internet access.
- NAT Gateway: Enables instances in private subnets to access internet resources.
- Network ACLs (NACLs): Enforce granular traffic control at subnet level.
- Route Tables: Manage traffic routing within and between subnets.
- Availability Zones: Public and private subnets span multiple AZs for high availability.

##### CI/CD Integration with Terraform and GitHub Actions:
Terraform the Infrastructure as Code (IaC) tool. GitHub Actions as the CI/CD tool.<br>
Workflow:
The workflow is triggered on code changes or manual events by developers. GitHub Actions initiates build and test processes to ensure code quality. Deployment artifacts are stored in an S3 bucket within the web tier.  Elastic Container Service (ECS) is updated with the new deployment package in the app tier.

##### Other Services: 
- Security: Effective security measures with WAF, Shield, Cognito, and private subnets.
- Scalability: Serverless components (Lambda, DynamoDB) scale automatically. ALBs and RDS can scale as needed. Other resources such as EC2 instances are within autoscaling groups in web, app and database tier where necessary.
- Cost Optimization: Serverless components incur costs only when used.

#### Link to architectural diagram of Hybrid three tier with serverless and CI/CD:
https://drive.google.com/file/d/16u1JkQX0JvSH5hihLGgRfYbh0wi5fw7E/view?usp=sharing


## __Serverless with CI/CD__
This architecture is consists of serverless resources. They include:
- Route 53: Global DNS service for routing traffic to the nearest CloudFront distribution.
- CloudFront: Content delivery network for caching static content and reducing latency.
- API Gateway: API management service for handling web and mobile requests, routing, authentication, and authorization.
- Lambda: Serverless compute service for executing application logic and accessing DynamoDB.
- DynamoDB: NoSQL database for storing application data.
- Cognito: User authentication and authorization service.
- Amplify: Front-end development framework for building web and mobile applications.
- Terraform: Infrastructure as code (IaC) tool for managing cloud resources.
- GitHub Actions: CI/CD pipeline tool for automating deployments.
  
#### Architecture Overview:
##### Domains:
* Public Domain:
Route 53, CloudFront, API Gateway, Lambda (exposed as API endpoints).
* Private Domain:
DynamoDB

##### Flow Pattern:
User accesses the application through a URL or mobile app. A DNS query goes to Route 53 then to CloudFront. CloudFront caches static content and forwards requests to API Gateway. The API Gateway routes web requests to Lambda functions for logic execution. It routes mobile requests to AWS Amplify backend endpoints which in turn interact with Lambda. The API Gateway also routes health check requests to Lambda Alpha for health monitoring. Lambda functions interact with DynamoDB for data operations. API Gateway handles user authentication and authorization using Cognito. With regards to the frontend development of the application, AWS Amplify provides tools and services for building web and mobile UIs.

##### Multi-Region Active-Active:
The entire architecture is replicated in multiple regions for high availability and disaster recovery. Route 53 utilizes latency-based routing to direct users to the nearest region.

##### Networking:
- Public Subnets:
  + API Gateway: Needs direct internet access to receive and route requests.
  + Lambda for Health Check: Requires internet access to perform external health checks.
  + CloudFront: Essential for public content delivery.
  + Cognito User Pools: Expose endpoints for user authentication.
- Private Subnets:
  + Lambda for Logic: Unless it needs to call external services but will be kept in a private subnet for security.
  + DynamoDB: Enhanced security by restricting public access.

##### CI/CD Integration with Terraform and GitHub Actions:
Terraform as the Infrastructure as Code (IaC) and GitHub Actions as the CI/CD tool.<br>
Workflow:
Triggered by code changes or manual events by developers. GitHub Actions initiates build and test processes to ensure code quality. Deployment artifacts are stored and Terraform is used to deploy changes to the infrastructure in a version-controlled manner. AWS Amplify smoothly integrates Terraform and GitHub Actions for continuous deployment of frontend and backend code.

##### Other Services: 
- Security: Effective security measures with WAF, Shield, Cognito, and private subnets.
- Simple Storage Service (S3): For static content storage.
- Simple Notification Service/ Simple Queue Service (SNS/SQS): For asynchronous messaging and event driven interactions.

#### Link to architectural diagram of Serverless with CI/CD: 
https://drive.google.com/file/d/1_yn3zV-wSbgivOPWraigTM3NqfjC69r6/view?usp=sharing

