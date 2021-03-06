# Demonstration of Full Stack Provisioning and CI/CD Pipeline in AWS [![CircleCI status](https://circleci.com/gh/CircleCI-Public/circleci-demo-aws-ecs-ecr.svg "CircleCI status")](https://circleci.com/gh/CircleCI-Public/circleci-demo-aws-ecs-ecr)

This project builds a Docker image containing a sample flask application on [CircleCI](https://circleci.com), pushes it to an Amazon Elastic Container Registry (ECR), and then deploys to Amazon Elastic Container Service (ECS) using AWS Fargate.

Code changes to the master branch of this repo will result in deployment to the ECS cluster.

This is designed to be separated by long running Git branches. Groups will have targetted access to these branches such as Dev, Staging and Productuion. For example, QA may only access Staging, developers may access only Dev and DevOps would access all platforms.

A Dockerfile is used to create an image, which is uploaded to the AWS Container Registry (ECR) which is then used by ECS and Fargate to perform compute services.

The stack has default CloudWatch monitoring.

## Deploy to AWS ECS from ECR via CircleCI

### Prerequisites and AWS Stack Deployment
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
4. Create the project on [CircleCI](https://circleci.com) (creating an account if necessary), and link it to this repository.
5. FYI: Run `terraform destroy` to destroy most of the created AWS resources but in case of lingering undeleted resources, check the [AWS Management Console](https://console.aws.amazon.com/) to see if there are any remaining undeleted resources. In particular, please check the ECS, CloudFormation and VPC pages.

![Simplified Architecture Diagram](https://raw.githubusercontent.com/douglalonde/benchsci-ci-cd-stack-demo/master/aws_ci_cd_pipeline_architecture.png)
### Configure environment variables on CircleCI
The following [environment variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project) must be set for the project on CircleCI via the project settings page, before the project can be built successfully.

### Add Application Requirments or Edit App Deployment Details

- Edit the Dockerfile to alter the application configuration.
- Edit .circleci/config.yml to change stack deployment options.
- Edit requirements.txt to add additional packages

| Variable                       | Description                                               |
| ------------------------------ | --------------------------------------------------------- |
| `AWS_ACCESS_KEY_ID`            | Used by the AWS CLI                                       |
| `AWS_SECRET_ACCESS_KEY `       | Used by the AWS CLI                                       |
| `AWS_DEFAULT_REGION`           | Used by the AWS CLI. Example value: "us-east-1" (Please make sure the specified region is supported by the Fargate launch type)                          |
| `AWS_ACCOUNT_ID`               | AWS account id. This information is required for deployment.                                   |
| `AWS_RESOURCE_NAME_PREFIX`     | Prefix that some of the required AWS resources are assumed to have in their names. The value should correspond to the `aws_resource_prefix` variable value in `terraform_setup/terraform.tfvars`.                             |

### Add Additional Compute Resources
To increase the robustness of this stack or limit Fargate costs, edit the default DesiredCount count value in modules/cloudformation-templates/public-service.yml.

## TO DO
- Ensure security groups are applied to the correct components, test.
- Extract common values to variable files.
- Augment health checks and monitoring. 
- Implement CloudWatch, consider Prometeus, Zabbix etc.
- Implement centralized logging:
https://github.com/awslabs/aws-centralized-logging
https://github.com/awslabs/aws-centralized-logging
- app.py logging to be redirected to STDOUT for instance level log access.
- Implement infrastructure change set and release method.

## Useful Links & References
- https://circleci.com/orbs/registry/orb/circleci/aws-ecr
- https://circleci.com/orbs/registry/orb/circleci/aws-ecs
- https://github.com/CircleCI-Public/aws-ecr-orb
- https://github.com/CircleCI-Public/aws-ecs-orb
- https://github.com/awslabs/aws-cloudformation-templates
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html
- Secrets management: no immediate tie in found for Vault (keep looking). Secrets currently in .gitignore files)
