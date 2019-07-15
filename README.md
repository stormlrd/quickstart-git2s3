# quickstart-git2s3 v2
## Git webhooks with AWS services
### Linking your Git repository to Amazon S3 and AWS services for continuous code integration, testing, and deployment 

This Quick Start deploys HTTPS endpoints and AWS Lambda functions for implementing webhooks, to enable event-driven integration between Git services and Amazon Web Services (AWS) on the AWS Cloud, including private git servers hosted internally in a VPC or on premise.

After you deploy the Quick Start, you can set up a webhook that uses the endpoints to create a bridge between your Git repository and AWS services like AWS CodePipeline and AWS CodeBuild. With this setup, builds and pipeline executions occur automatically when you commit your code to a Git repository, and your code can be continuously integrated, tested, built, and deployed on AWS with each change. 

The Quick Start includes an AWS CloudFormation template that automates the deployment. You can also use the AWS CloudFormation template as a starting point for your own implementation.

![Quick Start architecture for implementing webhooks on AWS](https://d0.awsstatic.com/partner-network/QuickStart/datasheets/git-to-s3-webhooks-architecture-on-aws.png)

For implementation details, deployment instructions, and customization options, see the [deployment guide](https://fwd.aws/QQBRr).

To post feedback, submit feature ideas, or report bugs, use the **Issues** section of this GitHub repo.
If you'd like to submit code for this Quick Start, please review the [AWS Quick Start Contributor's Kit](https://aws-quickstart.github.io/). 

##Public vs Private
This solution can be built one of two ways, Public or Private.

The original quick start delivers an API Gateway hosted on the internet. This is the "public" solution.

In order to use this concept internally when using a private bitbucket for example, a few changes need to be made to the original public solution.
1 - The API Gateway needs to be set to private an fronted by PrivateLink
2 - The Lambda function(s) need to become become network bound

###Controlling Public vs Private
Entering a PrivateServerIP parameter will cause MakePrivateSolution condition to be set to true.
Else if the PrivateServerIP is null, the condition MakePublicSolution is set to true.

MakePrivateSolution and MakePublicSolution condtions control the Public vs Private build mode of this template.

If you do not enter a Private Server IP Address the original quick start public solution is built with the enhanced feature of encrypting the bucket.

##Private Solution Details
When the solution is being set to Private by entering a Private Server IP Adrress the following occurs:
1 - The API Gateway becomes private & fronted by a PrivateLink interface eni
2 - The Lambda function(s) become network bound

Because of this the following is required:
1 - Security groups for the Lambda Function(s)
2 - VPC Endpoints for both S3 gateway and API Gateway Interface
3 - A Security group for the API Gateway Interface

The full list of resources controlled by the MakePrivateSolution condition being set to true are:
- PrivateGitPullLambda
- PrivateLambdaSecurityGroup
- PrivateAPIEndpointSecurityGroup
- PrivateLambdaSecurityGroupIngressRule
- PrivateWebHookRole
- PrivateWebHookApi
- PrivateWebHookApiDeployment
- PrivateWebHookApiProdStage

###Endpoint Creation
It could be that you have already have VPC Endpoints and are not permitted to create more.

If this is the case, the parameter CreateEndpoints is used to control the creation of the endpoints and should be set to false.

Difficulties arise due to route table entries and security group rules required to ensure that the endpoints are usable. 
Using CloudFormation conditions helps to work around this but requires an understanding of the logic for the input parameters and what they control.

If the CreateEndpoints parameter is true then the condition CreatePrivateVPCEndpoints is set to true and the following resources are created:
- PrivateAPIVPCPrivateLinkEndPoint
- PrivateS3VPCEndPoint
- PrivateAPIEndpointSecurityGroup
- PrivateLambdaSecurityGroupEgressRuleB

The following parameter is required to be entered as part of the S3 Gateway Endpoint Creation (as this QuickStart does not contain VPCs / Subnets / Route Tables they are assumed to pre-exist):
- PrivateLambdaSubnetRouteTableID

If CreatePrivateVPCEndpoints condition is set to false (i.e.CreateEndpoints parameter is false) the following **parameters** are required to be populated when building the stack:
- S3PrefixListID
- APIGatewayEndpointSGID

PrivateLambdaSubnetRouteTableID is the Route Table ID of the route table servicing the subnet you selected for parameter PrivateLambdaSubnetID.
S3PrefixListID is the pre-existing S3 Gateway Endpoint ID on the VPC you are deploying into. You will have to find this before creating the stack.
APIGatewayEndpointSGID is the pre-existing API Gateway Interface security group id.

Either way the following security group get created:
- PrivateLambdaSecurityGroup

##Conditions
###MakePrivateSolution
**Purpose**: To make the solution being built a private one for enterprises using a private bit bucket server for example
**Logic**: If PrivateServerIP is not null

###MakePublicSolution
**Purpose**: To use the original quickstart solution that is hosting the API Gateway on the internet
**Logic**: If PrivateServerIP is null

###CreateGitPullResources
**Purpose**: To control if the resources that support the Git Pull process are created or not.
**Logic**: If CreateGitPull is true then CreateGitPullResources is set to true.

###CreatePrivateVPCEndpoints
**Purpose**: To control if VPC Endpoints are to be created by the template or not
**Logic**: If PrivateServerIP is not null and CreateEndpoints is set to true

###EndPointsPreExist 
**Purpose**: To Control PrivateLambdaSecurityGroupEgressRuleA being created which has the Ref Lookup for the S3PrefixListID parameter because the endpoint already exists
**Logic**: If S3PrefixListID is blank and CreateEndpoints is false then EndPointsPreExist is true

###S3VPCEndPointAuto 
**Purpose**: To Control PrivateLambdaSecurityGroupEgressRuleB beign created which has the Ref Lookup for the PrivateS3VPCEndPoint resource  because the endpoint is being created
**Logic**: If CreateEndpoints is true then S3VPCEndPointAuto is true