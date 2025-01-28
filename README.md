# Bedrock-Flask Dev Env Quickstart

This quickstart spins up a simple dev server in a private subnet (in a preexisting VPC and subnet), and provides a pattern for integrating with OpenVPN Access Server for secure access.

## Prerequisites

Before you begin, ensure you have the following:

1. An AWS account
2. Terraform installed on your local machine (version 0.12 or later)
3. AWS CLI installed and configured with your credentials
5. Basic knowledge of AWS, Terraform, and command-line operations
6. Subscription to the DevOpser Flask AMI (see instructions below)

## Configuring AWS Credentials

Before running Terraform, you need to configure your AWS credentials. There are several ways to do this, but for this quickstart, we'll use the AWS CLI method:

1. Install the AWS CLI if you haven't already. You can download it from the [official AWS CLI page](https://aws.amazon.com/cli/).

2. Open a terminal and run:
   ```
   aws configure
   ```

3. You'll be prompted to enter your AWS Access Key ID, Secret Access Key, default region name, and default output format. Enter these details as provided by your AWS account administrator.

4. Your credentials will be stored in `~/.aws/credentials` (Linux/macOS) or `%UserProfile%\.aws\credentials` (Windows).

## IAM Permissions

Ensure that the IAM user or role you're using has sufficient permissions to create and manage the required AWS resources. Here's a demo policy that grants the necessary permissions for this quickstart:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "elasticloadbalancing:*",
                "route53:*",
                "acm:*",
                "iam:*",
                "secretsmanager:*",
                "logs:*",
                "autoscaling:*",
                "cloudwatch:*",
                "vpc:*",
                "bedrock:*",
            ],
            "Resource": "*"
        }
    ]
}
```

To use this policy:

1. Go to the AWS IAM console.
2. Create a new policy by navigating to "Policies" and clicking "Create policy".
3. In the JSON tab, paste the above policy.
4. Review and create the policy, giving it a name like "BedrockFlaskQuickstartPolicy".
5. Attach this policy to the IAM user or role you're using for this quickstart.

**Note**: This is a broad policy for demonstration purposes. In a production environment, you should follow the principle of least privilege and grant only the specific permissions needed for your use case.

## Verifying Your AWS Configuration

To verify that your AWS credentials are correctly configured and have the necessary permissions:

1. Run the following AWS CLI command:
   ```
   aws sts get-caller-identity
   ```

2. This should return your AWS account ID, IAM user/role ID, and ARN. If you see this information, your credentials are correctly set up.

3. To check your permissions, you can use the IAM Policy Simulator in the AWS Console, or try a simple AWS CLI command that uses one of the required services, such as:
   ```
   aws ec2 describe-regions
   ```

If these commands work without errors, your AWS configuration should be ready for running this quickstart.

## Subscribing to the DevOpser Flask AMI

Before you can use this quickstart, you need to subscribe to the DevOpser Flask AMI from the AWS Marketplace. Follow these steps:

1. Visit the [DevOpser Flask AMI page on AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-tti62q7ulbcoq)

2. Click on the "Continue to Subscribe" button.

3. Review the terms and conditions, then click "Accept Terms".

4. Once your subscription is active, click on "Continue to Configuration".

5. On the configuration page:
   - Select the AWS region closest to your users to reduce latency (e.g., North Virginia).
   - copy the AMI-id and use this in the configuration for the `bedrockflask_ami_id` variable in your `terraform.tfvars` file or upload as a variable in Terraform Cloud.

6. Your subscription is now active, and you can use the AMI ID in your Terraform configuration.

## Getting Started

1. Clone this repository to your local machine.
2. Navigate to the project directory.

## Configuration

Create a `terraform.tfvars` file in the project directory to set the required variables. Here's a template with explanations:

```hcl
aws_region         = "us-east-1"              # The AWS region to deploy resources
bedrockflask_ami_id = "ami-xxxxxxxxxxxxxxxxx" # AMI ID from the subscription process
key_name           = "your-key-pair-name"     # Your EC2 key pair name
your_ip_address    = "x.x.x.x"               # Your IP address for SSH access
flask_secret_key   = "your-flask-secret-key"  # A secret key for Flask
vpc_id             = "vpc-xxxxxxxxxxxxxxxxx"  # VPC ID for a preexisting VPC
domain_name        = "example.com"           # The domain name for the application in Route53
subdomain          = "app"                   # The subdomain for the application
dev_db_name        = "myappdb"              # Name of the production database
POSTGRES_USER      = "dbuser"               # Username for the PostgreSQL database
POSTGRES_PASSWORD  = "dbpassword"           # Password for the PostgreSQL database
POSTGRES_PORT      = "5432"                 # Port number for the PostgreSQL database
mail_password      = "mailpassword"         # Password for the mail server
email_for_mail_server = "noreply@example.com" # Default sender email address
additional_secrets = "{}"                   # Additional secrets in JSON format
admin_users        = "{}"                   # List of admin users in JSON format
```

### Variable Values

- `aws_region`: Choose the AWS region where you want to deploy the resources. This should match the region you selected during the AMI subscription process.
- `bedrockflask_ami_id`: Use the AMI ID you noted down during the subscription process.
- `key_name`: Create an EC2 key pair in your AWS account and provide its name.
- `your_ip_address`: Your IP address for SSH access in x.x.x.x format.
- `flask_secret_key`: A secret key for Flask. Generate a strong, random string for this.
- `vpc_id`: VPC ID for a preexisting VPC.
- `domain_name`: The domain name for your application (must be in Route53).
- `subdomain`: The subdomain for your application.
- `dev_db_name`: Name of your PostgreSQL database.
- `POSTGRES_USER`: Username for the PostgreSQL database.
- `POSTGRES_PASSWORD`: Password for the PostgreSQL database.
- `POSTGRES_PORT`: Port number for the PostgreSQL database (default: 5432).
- `mail_password`: Password for the mail server.
- `email_for_mail_server`: Default sender email address for the mail server.
- `additional_secrets`: Additional secrets for the application in JSON format (default: "{}").
- `admin_users`: List of admin users for the application in JSON format (default: "{}").

You can also customize the subnet tag settings if needed (defaults shown below):
- `public_subnet_tag_key`: "Tier" (default)
- `public_subnet_tag_value`: "Public" (default)
- `private_subnet_tag_key`: "OS" (default)
- `private_subnet_tag_value`: "Ubuntu" (default)

**Note on Variable Handling and Terraform Cloud**: While using a `terraform.tfvars` file is convenient for local development, it's not the most secure method for handling sensitive variables in a production environment. For enhanced security and better secret management, we strongly recommend using Terraform Cloud. Here's how to set it up:

1. Fork this repository to your own GitHub account.

2. Sign up for a Terraform Cloud account at [https://app.terraform.io/signup/account](https://app.terraform.io/signup/account) if you haven't already.

3. In Terraform Cloud, create a new workspace and choose "Version control workflow" to connect it to your forked GitHub repository.

4. In your workspace settings, navigate to the "Variables" section. Here, you can add all the variables from the `terraform.tfvars` file as Terraform variables. For sensitive variables like `POSTGRES_PASSWORD` and `flask_secret_key`, make sure to mark them as sensitive.

5. To authenticate with AWS, you can use [dynamic provider credentials](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/dynamic-provider-credentials/aws-configuration). 

6. With this setup, you can now run your Terraform operations directly from Terraform Cloud. It will use the variables you've set and the AWS credentials you've provided.

Using Terraform Cloud provides several benefits:
- Secure storage of sensitive variables and state files
- Collaboration features for team environments
- Consistent execution environment
- Integration with version control systems
- Automated runs based on repository changes

You can learn more about Terraform Cloud [here](https://www.terraform.io/cloud).

## Deployment Steps from local

1. Initialize Terraform:
   ```
   terraform init
   ```

2. Review the planned changes:
   ```
   terraform plan
   ```

3. Apply the configuration:
   ```
   terraform apply
   ```

4. Confirm the changes by typing `yes` when prompted.

5. Wait for the deployment to complete. This may take several minutes.

6. Once complete, Terraform will output the URL of your application.

## Accessing Your Application

After successful deployment, you can access your application at:

```
https://<subdomain>.<domain_name>
```

Subdomain and domain_name are variables - please note the domain should be hosted on AWS Route53.

## Terraform Outputs

After a successful deployment, Terraform will display several outputs that provide important information about your infrastructure. You can also retrieve these outputs at any time by running `terraform output`. Here are the key outputs:

- `application_url`: The URL of your Bedrock Flask application
- `alb_dns_name`: The DNS name of the Application Load Balancer
- `ec2_instance_ids`: The IDs of the EC2 instances
- `ec2_private_ips`: The private IP addresses of the EC2 instances
- `alb_security_group_id`: The ID of the ALB security group
- `ec2_security_group_id`: The ID of the EC2 instances' security group
- `acm_certificate_arn`: The ARN of the ACM certificate
- `route53_zone_id`: The Zone ID of the Route 53 hosted zone
- `rendered_env_template`: A rendered .env template for local development (sensitive value)

To get the rendered .env template for local development, you can run:
```bash
terraform output rendered_env_template
```

This will provide you with a rendered .env template for local development. To create the .env file:

1. Run `terraform output -raw rendered_env_template > .env`
2. The command will create a .env file in your current directory that you can use for local development.

**Note**: The .env file contains secret names. You'll need to replace these with actual secret values from AWS Secrets Manager for local development.

## Rendering your .env File with "/n" notation can be achieved in a few ways (pick one you like)

We've provided multiple methods to make this process straightforward so you dont need to make this file manually:

### Option 1: Using an AI Assistant (Recommended)

1. Run `terraform output -raw rendered_env_template`
2. Copy the entire output (including any quotation marks)
3. Visit your preferred AI assistant (Claude, ChatGPT, etc.)
4. Use the following prompt:
   ```
   Please convert this escaped string into a proper .env file format, removing all \n and replacing them with actual newlines:

   [paste your terraform output here]
   ```
5. Copy the AI-generated .env file contents
6. Save to a `.env` file in your project root

### Option 2: Using Unix Command Line 

For Unix/Linux users, use `echo` with the `-e` flag:

```bash
terraform output -raw rendered_env_template | sed 's/\\n/\n/g' > .env
```

### Verification

After creating your `.env` file, verify its contents:

```bash
cat .env
```

You should see properly formatted environment variables, each on its own line:

```
FLASK_APP=run
FLASK_ENV=development

FLASK_SECRET_NAME=${flask_secret_name}
REDIS_URL=redis://localhost:6379/0
REGION=us-east-1

DB_NAME_SECRET_NAME=${db_name_secret_name}
DB_USER_SECRET_NAME=${db_user_secret_name}
DB_PASSWORD_SECRET_NAME=${db_password_secret_name}
DB_HOST_SECRET_NAME=${db_host_secret_name}
DB_PORT_SECRET_NAME=${db_port_secret_name}

MAIL_SERVER=${mail_server}
MAIL_PORT=${mail_port}
MAIL_USE_TLS=${mail_use_tls}
MAIL_USERNAME=${email}
MAIL_DEFAULT_SENDER=${email}
MAIL_PASSWORD_SECRET_NAME=${mail_password_secret_name}

ADDITIONAL_SECRETS=${additional_secrets}
ADMIN_USERS_SECRET_NAME=${admin_users_secret_name}
```

### Troubleshooting

- Ensure you're using the correct method for your operating system
- If you see literal `\n` characters, double-check your conversion method

**Pro Tip**: Never commit your `.env` file to version control. Add it to your `.gitignore` to protect sensitive information.

## Cleaning Up

To avoid incurring unnecessary costs, remember to destroy the resources when you're done:

```
terraform destroy
```

Confirm the destruction by typing `yes` when prompted.

## Important Notes

1. **Costs**: This quickstart creates AWS resources that may incur costs. Based on current estimates, the resources in this dev configuration costs $31.75/month (if you use a t3.medium to develop on). Always review the AWS pricing for EC2 instances, Application Load Balancers, and associated services before deploying.

2. **Security**: While this quickstart provides a basic secure setup, it's recommended to implement additional security measures for production use.

3. **Dependencies**: Make sure all dependencies (Terraform, AWS CLI) are correctly installed and configured before starting.

## Support

For any questions or assistance with this quickstart, please contact DevOpser at [info@devopser.io](mailto:info@devopser.io) or [join our Slack community](https://join.slack.com/t/devopser-workspace/signup). We're here to help you implement more advanced configurations, address any issues you may encounter, or discuss how we can help optimize your deployment for enhanced security and cost-effectiveness. It is also open source so feel free to submit a pull request with any changes and we will review them.

## Disclaimer

This quickstart is provided as-is. Always review and understand the code before deploying resources in your AWS account. DevOpser is not responsible for any costs incurred or security issues that may arise from using this quickstart.