## Architecture:
AWS Hybrid Three-Tier with Serverless Components and CI/CD: 
The architecture combines a traditional three-tier structure (web, app, database) with serverless components (Lambda functions) for flexibility and scalability.

## Flow Pattern:
* __User Request:__
- User accesses application via Route 53 DNS service.
- Route 53 directs request to CloudFront CDN for global content delivery.
  
* __External ALB Distribution:__
- CloudFront forwards request to an external Application Load Balancer (ALB) for traffic distribution. It also delivers static assets stored in S3 bucket.
- ALB routes request based on path and type:
  + API Requests: To API Gateway for handling RESTful APIs.
  + Web Requests: To web tier (S3) for static content.
 
* __Security and Authentication:__
- AWS WAF: Inspects and filters incoming traffic, blocking malicious requests.
- AWS Shield: Protects against Distributed Denial of Success (DDoS) attacks.
- AWS Cognito: Manages user authentication and authorization.
- IAM: Controls access to AWS resources based on defined policies.

* __API Gateway and Lambda Functions:__
- API Gateway invokes Lambda function lambda_a for authentication and authorization (using AWS Cognito).
- lambda_a may invoke lambda_b for database interactions via a private link, ensuring secure database access.
- lambda_a and lambda_b are separate functions for decoupled responsibilities. Lambda_a handles authentication, authorization, and initial processing while lambda_b,
  via a private link, communicates with RDS Proxy in the database tier for database-related activities.

* __Internal ALB and App Tier:__
- Exterior ALB routes dynamic content requests to internal ALB within a private subnet.
- Internal ALB distributes requests to ECS containers running the application logic.

* __Database Tier:__
- RDS Proxy offers secure, managed access to the database.
- Relational Database Service (RDS) host application database.
- Lambda function lambda_b interacts with the database through RDS Proxy.
- DynamoDB used for additional data storage needs.
  
* __Caching:__
- Redis for caching frequently accessed data, improving performance.

* __Autoscaling:__
- AWS services like Lambda and API Gateway automatically scale based on demand.
- ECS (Elastic Container Service) and EC2 (Elastic Compute Cloud) instances in the App Tier and other EC2 instances in web tier placed are in autoscaling groups to meet demand.

* __Security Group:__
- Resources within the web tier, app tier and database tier have associated security groups or access control mechanisms to control inbound and outbound traffic to resources.
   
* __CI/CD:__
- Code Commit and Testing:
  + Developers commit code to Git, and the code is pushed to the GitHub repository.
  + GitHub Actions trigger CI/CD upon the push event.
- Build and Test:
  + GitHub Actions build and test the application code.
- Deployment:
  + Deployment artifacts are stored in an S3 bucket within the Web Tier.
  + ECS in the App Tier is updated with the new deployment package.
- Terraform is the Infrastructure as Code (IaC).

* __Subnets:__
- Public Subnet: External ALB, API Gateway, CloudFront, WAF, Shield, Cognito, S3, DynamoDB, Route 53.
- Private Subnet: Internal ALB, ECS containers, RDS Proxy, Lambda functions (via private link), RDS, ElastiCache Redis.
