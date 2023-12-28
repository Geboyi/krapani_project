## Architecture:
AWS Hybrid Three-Tier with Serverless Components and CI/CD: 
The architecture combines a traditional three-tier structure (web, app, database) with serverless components (Lambda functions) for flexibility and scalability.

Requirements were to be able to handle mobile and web traffic:
Mobile traffic is handled by the API gateway while web traffic is handled by the ALB. 

The architecture is fronted by cloudfront which will route traffic to either ALB or API Gateway based on the source of the request. Once traffic gets into the system we can use a completely serverless solution. 

__Option One - Serverless__

For serverless, the lambda functions will be used. Lambda-a is a routing lambda whose job is simply routing - it does not perform business logic. It route to lambda-b. 

Lambda-b is the main business logic function which will interact with other parts of the system. It will be deployed to have connectivity into the VPC so it can access the RDS, and ElastiCache services. Since it is the main processing function it is also made a target of the ALB, so the ALB can route to it. 

Since lambda function scale well in the face of increasing traffic this solution design will be capable of handling large volume of user traffic. 

All static content will be served from S3 and dynamic content from the lambda function. Cloudfront will cache some of the content hence improving user experience.

Since it is supposed to be a global service with users all over the world I could consider using a database with global capabilities such as DynamoDB or Aurora. 
I may also want to use a global load balancer 

__Option two - Hybrid Solution__

In this option, a three tier architecture is also created in the VPC to handle the web traffic.


## Flow Pattern:
#### User Request:
- User accesses application via Route 53 DNS service.
- Route 53 directs request to CloudFront CDN for global content delivery.
   
#### Frontend:
- CloudFront forwards request to an external Application Load Balancer (ALB) or the API Gateway for traffic distribution. It also delivers static assets stored in S3 bucket.
- CloudFront routes request based on path and type:
  + API Requests: To API Gateway for handling RESTful APIs.
  + Web Requests: To exterior ALB for traffic distribution.
 
#### Security and Authentication:
- AWS WAF: Inspects and filters incoming traffic, blocking malicious requests.
- AWS Shield: Protects against Distributed Denial of Success (DDoS) attacks.
- AWS Cognito: Manages user authentication and authorization.
- IAM: Controls access to AWS resources based on defined policies.

#### API Gateway and Lambda Functions:
- API Gateway invokes Lambda function lambda_a for authentication and authorization (using AWS Cognito).
- lambda_a is a routing lambda. It routes to lambda_b, lambda_b is the main logic process function which interacts with the other systems.

#### Internal ALB and App Tier:
- Exterior ALB routes dynamic content requests to the lambda_b and internal ALB within a private subnet.
- Internal ALB distributes requests to ECS containers running the application logic.

#### Database Tier:
- RDS Proxy offers secure, managed access to the database.
- Relational Database Service (RDS) host application database.
- Lambda function lambda_b interacts with the database through RDS Proxy.
- DynamoDB used for additional data storage needs.
  
#### Caching:
- Redis for caching frequently accessed data, improving performance.

#### Autoscaling:
- AWS services like Lambda and API Gateway automatically scale based on demand.
- ECS (Elastic Container Service) and EC2 (Elastic Compute Cloud) instances in the App Tier and other EC2 instances in web tier placed are in autoscaling groups to meet demand.

#### Security Group:
- Resources within the web tier, app tier and database tier have associated security groups or access control mechanisms to control inbound and outbound traffic to resources.
   
#### CI/CD:
- Code Commit and Testing:
  + Developers commit code to Git, and the code is pushed to the GitHub repository.
  + GitHub Actions trigger CI/CD upon the push event.
- Build and Test:
  + GitHub Actions build and test the application code.
- Deployment:
  + Deployment artifacts are stored in an S3 bucket within the Web Tier.
  + ECS in the App Tier is updated with the new deployment package.
- Terraform is the Infrastructure as Code (IaC).

#### Subnets:
- Public Subnet: External ALB, API Gateway, CloudFront, WAF, Shield, Cognito, S3, DynamoDB, Route 53.
- Private Subnet: Internal ALB, ECS containers, RDS Proxy, Lambda functions (via private link), RDS, ElastiCache Redis.

#### Link to architectural diagram:
#### Hybrid three tier with serverless
https://drive.google.com/file/d/16u1JkQX0JvSH5hihLGgRfYbh0wi5fw7E/view?usp=sharing
#### Serverless 
https://drive.google.com/file/d/1_yn3zV-wSbgivOPWraigTM3NqfjC69r6/view?usp=sharing

