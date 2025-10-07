# PDF Processing AWS Infrastructure

# Austin was here in VS Code

This project builds an AWS infrastructure using AWS CDK (Cloud Development Kit) to split a PDF into chunks, process the chunks via AWS Step Functions, and merge the resulting chunks back using ECS tasks. The infrastructure also includes monitoring via CloudWatch dashboards and metrics for tracking progress.

## Prerequisites

Before running the AWS CDK stack, ensure the following are installed and configured:

1. **AWS Bedrock Access**: Ensure your AWS account has access to the Nova pro model in Amazon Bedrock.
   - [Request access to Amazon Bedrock](https://console.aws.amazon.com/bedrock/) through the AWS console if not already enabled.

2. **Adobe API Access** - An enterprise-level contract or a trial account (For Testing) for Adobe's API is required.

   - [Adobe PDF Services API](https://acrobatservices.adobe.com/dc-integration-creation-app-cdn/main.html) to obtain API credentials.
   
4. **Python (3.7 or later)**  
   - [Download Python](https://www.python.org/downloads/)  
   - [Set up a virtual environment](https://docs.python.org/3/library/venv.html)  
     ```bash
     python -m venv .venv
     source .venv/bin/activate  # For macOS/Linux
     .venv\Scripts\activate     # For Windows
     ```
   - Also ensure that if you are using windows to confirm the python path in cmd before deploying. That can be done by running:
     ```bash
     where python
     ```

5. **AWS CLI**: To interact with AWS services and set up credentials.

   - [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
     
6. **npm**  
   - npm is required to install AWS CDK. Install npm by installing Node.js:  
     - [Download Node.js](https://nodejs.org/) (includes npm).  
   - Verify npm installation:  
     ```bash
     npm --version
     ```
7. **AWS CDK**: For defining cloud infrastructure in code.
   - [Install AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html)  
     ```bash
     npm install -g aws-cdk
     ```

8. **Docker**: Required to build and run Docker images for the ECS tasks.  
   - [Install Docker](https://docs.docker.com/get-docker/)  
   - Verify installation:  
     ```bash
     docker --version
     ```

9. **AWS Account Permissions**  
   - Ensure permissions to create and manage AWS resources like S3, Lambda, ECS, ECR, Step Functions, and CloudWatch.  
   - [AWS IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
   - Also, For the ease of deployment. Create a IAM user in the account you want to deploy to and attach adminstrator access to that user and use the Access key and Secret key for that user.

## Directory Structure

Ensure your project has the following structure:

```
├── app.py (Main CDK app)
├── lambda/
│   ├── split_pdf/ (Python Lambda for splitting PDF)
│   └── java_lambda/ (Java Lambda for merging PDFs)
├── docker_autotag/ (Python Docker image for ECS task)
└── javascript_docker/ (JavaScript Docker image for ECS task)
|__ client_credentials.json (The client id and client secret id for adobe)
```

## Setup and Deployment

1. **Clone the Repository**:
   - Clone this repository containing the CDK code, Docker configurations, and Lambda functions.
     
2. **Set Up Your Environment**:
   - Configure AWS CLI with your AWS account credentials:
     ```bash
     aws configure
     ```
   - Make sure the region is set to
     ```
     us-east-1
     ```
     
3. **Set Up CDK Environment**:
   - Bootstrap your AWS environment for CDK (run only once per AWS account/region):
     ```
     cdk bootstrap
     ```
     
4. **Create Adobe API Credentials**:
   - Create a file called `client_credentials.json` in the root directory with the following structure:
     ```json
     {
       "client_credentials": {
         "PDF_SERVICES_CLIENT_ID": "<Your client ID here>",
         "PDF_SERVICES_CLIENT_SECRET": "<Your secret ID here>"
       }
     }
     ```
   - Replace <Your Client ID here> and <Your Secret ID here> with your actual Client ID and Client Secret provided by Adobe and not the whole file.

5. **Upload Credentials to Secrets Manager**:
   - Run this command in the terminal of the project to push the secret keys to secret manager:
   - For Mac
     ```
     aws secretsmanager create-secret \
         --name /myapp/client_credentials \
         --description "Client credentials for PDF services" \
         --secret-string file://client_credentials.json
     ```
   - For Windows
     ```bash
     aws secretsmanager create-secret --name /myapp/client_credentials --description "Client credentials for PDF services" --secret-string file://client_credentials.json
     ```
   - Run this command if you have already uploaded the keys earlier and would like to update the keys in secret manager.
   - For Mac:
     ```
        aws secretsmanager update-secret \
       --secret-id /myapp/client_credentials \
       --description "Updated client credentials for PDF services" \
       --secret-string file://client_credentials.json
     ```
   - For Windows:
     ```bash
     aws secretsmanager update-secret --secret-id /myapp/client_credentials --description "Updated client credentials for PDF services" --secret-string file://client_credentials.json
     ```
6. **Install the Requirements**:
   - For both Mac and Windows
   - ```bash
     pip install -r requirements.txt
     ```
   
8. **Connect to ECR**:
   - Ensure Docker Desktop is running, then execute:
     ```
     aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
     ```
  
9. **Set a environment variable once for deployment**
   - An environment variable needs to be set before deployment. This step ensures compatibility and prevents deployment issues.
   - For additional guidance or if you encounter any deployment issues, please refer to [Troubleshooting](#troubleshooting) section.
   - For Mac,
     ```
     export BUILDX_NO_DEFAULT_ATTESTATIONS=1   
     ```
   - For Windows,
     ```
     set BUILDX_NO_DEFAULT_ATTESTATIONS=1
     ```
  
10. **Deploy the CDK Stack**:
   - Deploy the stack to AWS:
     ```
     cdk deploy
     ```

## Usage

Once the infrastructure is deployed:

1. Create a `pdf/` folder in the S3 bucket created by the CDK stack.
2. Upload a PDF file to the `pdf/` folder in the S3 bucket.
3. The process will automatically trigger and start processing the PDF.

## Monitoring

### PDF-to-PDF Solution
- **CloudWatch Dashboard**: Automatically created during deployment
- **Step Functions Console**: Monitor workflow executions
- **ECS Console**: Track container task status

### PDF-to-HTML Solution
- **Lambda Logs**: `/aws/lambda/Pdf2HtmlPipeline`
- **S3 Events**: Monitor file processing status
- **CloudWatch Metrics**: Track function performance

## Troubleshooting

### Common Issues

**AWS Credentials**
- Ensure AWS CLI is configured with appropriate permissions
- Verify access to required AWS services (S3, Lambda, ECS, Bedrock)

**Service Limits**
- Check AWS service quotas if deployment fails
- Request additional Elastic IPs if needed: [EC2 Service Quotas](https://us-east-1.console.aws.amazon.com/servicequotas/home/services/ec2/quotas)

**Build Failures**
- Check CodeBuild console for detailed error messages
- Verify all prerequisites are met
- Ensure Docker is available for PDF-to-HTML deployments

### Solution-Specific Troubleshooting

**PDF-to-PDF Issues**
- Verify Adobe API credentials are correct and active
- Check CloudWatch logs for Lambda functions and ECS tasks
- Ensure NOVA_PRO Bedrock model access is granted

**PDF-to-HTML Issues**
- Verify Bedrock Data Automation permissions
- Check Lambda function logs in CloudWatch
- Ensure Docker image was pushed to ECR successfully

### Getting Help

- Check build logs in CodeBuild console
- Review CloudWatch logs for runtime issues
- Verify all prerequisites are met
- For deployment issues, refer to: [CDK GitHub Issue](https://github.com/aws/aws-cdk/issues/30258)
- For additional troubleshooting: [Troubleshooting Guide](docs/TROUBLESHOOTING_CDK_DEPLOY.md)
- Contact support: **ai-cic@amazon.com**

## Contributing

Contributions to this project are welcome. Please fork the repository and submit a pull request with your changes.

---
