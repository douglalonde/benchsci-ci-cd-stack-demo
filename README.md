# Demonstration of Full Stack Provisioning and CI/CD Pipeline in AWS [![CircleCI status](https://circleci.com/gh/CircleCI-Public/circleci-demo-aws-ecs-ecr.svg "CircleCI status")](https://circleci.com/gh/CircleCI-Public/circleci-demo-aws-ecs-ecr)

## Deploy to AWS ECS from ECR via CircleCI
This project builds a Docker image on [CircleCI](https://circleci.com), pushes it to an Amazon Elastic Container Registry (ECR), and then deploys to Amazon Elastic Container Service (ECS) using AWS Fargate.

Code changes to the master branch of this repo will result in deployment to the ECS cluster.

## Prerequisites
### To Set up Required AWS Resources
The ECS cluster is created using Terraform.
1. Ensure [terraform](https://www.terraform.io/) is installed on your system.
2. Edit `terraform_setup/terraform.tfvars` to fill in the necessary variable values (an Amazon account with sufficient privileges to create resources like an IAM account, VPC, EC2 instances, Elastic Load Balancer, etc is required). (Do not commit this file to a public repository after it has been populated with your AWS credentials - use .gitignore)
3. Use terraform to create the AWS resources
    ```
    cd terraform_setup_aws
    terraform init
    # Review the plan
    terraform plan
    # Apply the plan to create the AWS resources
    terraform apply
    ```
4. Run `terraform destroy` to destroy most of the created AWS resources but in case of lingering undeleted resources, check the [AWS Management Console](https://console.aws.amazon.com/) to see if there are any remaining undeleted resources. In particular, please check the ECS, CloudFormation and VPC pages.

### Configure environment variables on CircleCI
The following [environment variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project) must be set for the project on CircleCI via the project settings page, before the project can be built successfully.


| Variable                       | Description                                               |
| ------------------------------ | --------------------------------------------------------- |
| `AWS_ACCESS_KEY_ID`            | Used by the AWS CLI                                       |
| `AWS_SECRET_ACCESS_KEY `       | Used by the AWS CLI                                       |
| `AWS_DEFAULT_REGION`           | Used by the AWS CLI. Example value: "us-east-1" (Please make sure the specified region is supported by the Fargate launch type)                          |
| `AWS_ACCOUNT_ID`               | AWS account id. This information is required for deployment.                                   |
| `AWS_RESOURCE_NAME_PREFIX`     | Prefix that some of the required AWS resources are assumed to have in their names. The value should correspond to the `aws_resource_prefix` variable value in `terraform_setup/terraform.tfvars`.                             |

## Add Additional Compute Resources
To increase the robustness of this stack or limit Fargate costs, edit the default DesiredCount count value in modules/cloudformation-templates/public-service.yml.


## Useful Links & References
- https://circleci.com/orbs/registry/orb/circleci/aws-ecr
- https://circleci.com/orbs/registry/orb/circleci/aws-ecs
- https://github.com/CircleCI-Public/aws-ecr-orb
- https://github.com/CircleCI-Public/aws-ecs-orb
- https://github.com/awslabs/aws-cloudformation-templates
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html
