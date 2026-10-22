# Terraform Module 3 — Build AWS Infrastructure

Junior DevOps • Beginner-friendly • Practical, job-focused

In Module 2, you learned how to write Terraform configurations using resources, variables, locals, and outputs. Now we'll apply those concepts to real AWS infrastructure.

This module focuses on the infrastructure a junior DevOps engineer should understand: networking, EC2, security groups, storage, IAM basics, and troubleshooting.

## Module 3 checklist

1\. AWS provider and credentials

Connect Terraform to AWS safely

2\. VPC and networking

VPC, CIDR, subnets, internet gateway and routes

3\. Security groups

Control inbound and outbound traffic

4\. EC2 instance

AMI, instance type, key pair and user data

5\. EBS storage and tags

Understand disks and resource organization

6\. IAM basics

Roles, policies and least privilege

7\. S3 basics

Create a bucket with secure settings

8\. Troubleshooting and cleanup

Fix errors, review costs and destroy resources

0 of 8 marked complete

## Lesson 1: Connect Terraform to AWS

### 1. What is a provider?

Terraform itself does not directly manage every cloud platform. A provider is a plugin that lets Terraform communicate with a service such as AWS.

For example:

- AWS provider → AWS resources
- Local provider → Local files
- Azure provider → Azure resources

AWS example:

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

Here, `ap-south-1` is the AWS Mumbai Region.

The `required_providers` block declares the provider dependency. The `provider` block configures it.

### 2. How does Terraform authenticate to AWS?

Terraform needs AWS credentials and permissions to create resources.

For a local Ubuntu/WSL lab, the typical setup uses the AWS CLI.

Check whether it is installed:

```
aws --version
terraform -version
```

If the AWS CLI is configured, you can check which identity it uses:

```
aws sts get-caller-identity
```

If this command succeeds, it returns information about the AWS account and identity associated with your current credentials.

Security rules:

- Never put access keys directly into `.tf` files.
- Never commit credentials to GitHub.
- Use an IAM identity with only the permissions your lab requires.
- Avoid root-account credentials.

You don't need administrator access for every Terraform task, but the identity must have permission for the specific resources and actions you plan to use.

## Lesson 2: Understand the AWS architecture

Before writing Terraform code, understand what we're building.

VPC

Your isolated virtual network

Public subnet

A subnet whose route table can send traffic through an Internet Gateway

EC2 instance

A virtual server, controlled by a security group

Internet Gateway

Provides a route between the VPC and the internet when configured correctly

A public subnet alone does not make an EC2 instance publicly reachable. You also need the appropriate route, public addressing, and security rules.

### Key terms

| Term             | Simple meaning                                      |
| ---------------- | --------------------------------------------------- |
| VPC              | Your private virtual network in AWS                 |
| CIDR             | Defines an IP address range, such as `10.0.0.0/16`  |
| Subnet           | A smaller IP range inside a VPC                     |
| Route table      | Determines where network traffic goes               |
| Internet Gateway | Connects a VPC to the internet when routes allow it |
| Security group   | Virtual firewall for supported AWS resources        |
| EC2              | Virtual server                                      |
| AMI              | Template used to launch an EC2 instance             |
| EBS              | Persistent block storage for EC2                    |

## Your first practical check

Before we create AWS resources, let's check your setup so we don't repeat installation work or create unnecessary costs.

Run these commands in Ubuntu/WSL, one at a time:

```
terraform -version
aws --version
aws sts get-caller-identity
```

Reply with the results. You can redact account IDs and other identifying details.

- If all three work, we'll build the AWS configuration.
- If Terraform or AWS CLI is missing, we'll fix only the missing part.
- If AWS authentication fails, we'll troubleshoot credentials before creating resources.

Cost and safety: We will review the plan before applying anything. AWS resources can incur charges, so we'll use a controlled lab and destroy the resources when finished.
Great! Now let's create your first AWS resource using Terraform: a VPC.

## Module 3 — Lesson 4: Create a VPC

A VPC is your own isolated virtual network inside AWS. Later, we'll put subnets and an EC2 server inside it.

### Step 1: Update `main.tf`

Open your existing project:

```
cd ~/terraform-lab
nano main.tf
```

Keep your existing `terraform` and `provider` blocks. Add this resource below them:

```
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "terraform-learning-vpc"
  }
}
```

Save using `Ctrl+O`, press Enter, then `Ctrl+X`.

### Step 2: Understand the code

| Setting                | Meaning                                      |
| ---------------------- | -------------------------------------------- |
| `aws_vpc`              | AWS VPC resource type                        |
| `main`                 | Terraform's internal name                    |
| `10.0.0.0/16`          | IP address range for the VPC                 |
| `enable_dns_support`   | Enables DNS resolution support               |
| `enable_dns_hostnames` | Enables DNS hostnames in the VPC             |
| `tags`                 | Labels the resource so it's easy to identify |

### Step 3: Preview the change

Run:

```
terraform fmt
terraform validate
terraform plan
```

If the configuration is valid and your AWS setup has the necessary access, the plan should propose creating one VPC.

### Step 4: Apply

If the plan shows the expected VPC creation, run:

```
terraform apply
```

Review the plan and type `yes` when prompted.

Then verify:

```
terraform state list
```

You should see:

```
aws_vpc.main
```

You have now practised the Terraform workflow with a real AWS resource.

Cost and safety: A VPC itself generally has no hourly charge, but related resources we'll create later may incur charges. Keep track of the lab and destroy its resources when you're finished.

### Your turn

Run Steps 3 and 4. Tell me whether the VPC was created successfully or paste the error message (remove any sensitive information). We'll troubleshoot it before moving on to subnets.
# Terraform Module 3 — Complete AWS Infrastructure

We’ll cover all remaining lessons together, from EC2 to S3, IAM, troubleshooting, and cleanup. This is the junior DevOps practical version: one project, working Terraform code, commands, and interview concepts.

We’ll use your existing `~/terraform-lab` project and the Mumbai AWS region (`ap-south-1`).

## Complete module checklist

| Lesson | What you’ll learn                                         |
| ------ | --------------------------------------------------------- |
| 1–2    | VPC, public subnet, Internet Gateway, routes              |
| 3      | Security Groups                                           |
| 4      | EC2, AMI lookup, user data, Nginx                         |
| 5      | EBS, tags, storage basics                                 |
| 6      | S3 bucket, encryption, versioning, public access blocking |
| 7      | IAM roles, policies, least privilege                      |
| 8      | Terraform data sources and resource references            |
| 9      | Common errors, permissions, region issues, cost awareness |
| 10     | Verification and cleanup                                  |

Lessons 1–3 have already been taught. The previous apply results haven’t been verified here, so inspect your existing configuration and plan before applying anything again.

## Lesson 4 — EC2 + automatic Nginx installation

### 1. How it works

Internet

Internet Gateway + public route

Public subnet + Security Group

EC2 instance

Amazon Linux + Nginx web server

- AMI: The operating-system image used to launch an EC2 instance.
- Instance type: The virtual server's CPU and memory capacity.
- User data: A startup script that installs and configures software when the instance launches.
- EBS: Persistent block storage attached to an EC2 instance.
- Security Group: Controls allowed network traffic.

### 2. Add the EC2 configuration

Open the file:

```
cd ~/terraform-lab
nano main.tf
```

Keep your existing Terraform, provider, VPC, subnet, Internet Gateway, route table, association, and security group blocks. Add these blocks at the bottom.

Add the following code. It covers EC2, an automatically selected Amazon Linux AMI, an IAM role, and an encrypted root EBS volume.

```
# Find the latest Amazon Linux 2023 AMI in the selected region
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-2023.*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# IAM role for Systems Manager access
resource "aws_iam_role" "ec2_role" {
  name = "terraform-learning-ec2-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "ssm" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

resource "aws_iam_instance_profile" "ec2_profile" {
  name = "terraform-learning-ec2-profile"
  role = aws_iam_role.ec2_role.name
}

# EC2 web server
resource "aws_instance" "web" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t3.micro"
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.web.id]
  associate_public_ip_address = true
  iam_instance_profile        = aws_iam_instance_profile.ec2_profile.name

  user_data = <<-EOF
    #!/bin/bash
    dnf install -y nginx
    systemctl enable --now nginx
    echo "<h1>Hello from Terraform and Nginx!</h1>" > /usr/share/nginx/html/index.html
  EOF

  root_block_device {
    volume_size           = 8
    volume_type           = "gp3"
    encrypted             = true
    delete_on_termination = true
  }

  tags = {
    Name        = "terraform-web-server"
    Environment = "learning"
  }
}
```

### 3. Understand what you just configured

| Configuration                  | Purpose                                                          |
| ------------------------------ | ---------------------------------------------------------------- |
| `data "aws_ami"`               | Finds an existing AMI; it does not create an AMI.                |
| `aws_instance`                 | Creates the EC2 instance.                                        |
| `user_data`                    | Installs and starts Nginx during initial startup.                |
| `root_block_device`            | Configures the instance's root EBS volume.                       |
| `encrypted = true`             | Encrypts the EBS volume at rest.                                 |
| IAM role and instance profile  | Give the EC2 instance permissions to use AWS Systems Manager.    |
| `AmazonSSMManagedInstanceCore` | AWS-managed permissions needed for core SSM management features. |

The EC2 role does not grant general S3 access. That is intentional: follow least privilege and grant only the permissions the workload actually needs.

## Lesson 5 — S3, encryption and versioning

S3 stores objects such as application files, deployment artifacts, logs, and backups. Bucket names must be globally unique across AWS accounts.

### 1. Add a bucket-name variable

Add this block to `main.tf`:

```
variable "bucket_name" {
  description = "Globally unique lowercase S3 bucket name"
  type        = string
}
```

Create a file named `terraform.tfvars`:

```
nano terraform.tfvars
```

Add a unique bucket name of your own, using lowercase letters, numbers, and hyphens:

```
bucket_name = "replace-with-your-unique-bucket-name"
```

Replace the example with a name that is globally unique. Do not use the placeholder unchanged.

### 2. Add the S3 resources

```
resource "aws_s3_bucket" "learning" {
  bucket = var.bucket_name

  tags = {
    Name        = "terraform-learning-bucket"
    Environment = "learning"
  }
}

resource "aws_s3_bucket_public_access_block" "learning" {
  bucket = aws_s3_bucket.learning.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "learning" {
  bucket = aws_s3_bucket.learning.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "learning" {
  bucket = aws_s3_bucket.learning.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

What these resources do:

- `aws_s3_bucket`: creates the bucket.
- `aws_s3_bucket_versioning`: keeps versions when objects are updated or deleted; it is not a substitute for a backup.
- Encryption configuration: enables server-side encryption using S3-managed keys.
- Public access block: prevents public access through the bucket's public ACLs and policies.

The public access block is particularly important for private files and backups.

## Lesson 6 — Outputs and resource references

Add these outputs:

```
output "vpc_id" {
  value = aws_vpc.main.id
}

output "ec2_instance_id" {
  value = aws_instance.web.id
}

output "website_url" {
  value = "http://${aws_instance.web.public_ip}"
}

output "s3_bucket_name" {
  value = aws_s3_bucket.learning.bucket
}
```

Terraform automatically understands dependencies through references such as:

```
subnet_id = aws_subnet.public.id
```

This means the EC2 instance needs the subnet ID, so Terraform creates the subnet before the instance.

You usually do not need `depends_on` when a direct reference already expresses the dependency.

## Lesson 7 — Run and verify the complete project

Before applying, confirm your `main.tf` still contains the previous networking resources and security group, along with the new blocks.

Run each command in order:

```
cd ~/terraform-lab

terraform fmt
terraform validate
terraform plan
```

If validation succeeds, inspect the plan carefully.

You should expect resources for:

- VPC, subnet, Internet Gateway, route table and association
- Security group
- IAM role, policy attachment and instance profile
- EC2 instance and its EBS root volume
- S3 bucket, public access block, versioning and encryption

If Terraform proposes unexpectedly replacing or destroying existing resources, stop before applying and inspect the plan.

If everything looks correct:

```
terraform apply
```

Type `yes` only after reviewing the proposed changes.

### Verify the result

```
terraform state list
terraform output
```

Copy the `website_url` output and open it in your browser. If Nginx has finished installing and the networking is correct, you should see:

Hello from Terraform and Nginx!

If the site does not load immediately, user data may still be running. Check the EC2 instance status, public IP, route table, and security-group HTTP rule. Never open SSH to all internet addresses just to troubleshoot the website.

## Lesson 8 — Common errors and troubleshooting

| Error or problem                          | What to check                                                                                 |
| ----------------------------------------- | --------------------------------------------------------------------------------------------- |
| `No valid credential sources found`       | AWS CLI credentials or your configured AWS role.                                              |
| `UnauthorizedOperation` or `AccessDenied` | IAM permissions for the operation; don't solve this by granting administrator access blindly. |
| AMI not found                             | AWS region, AMI filters, architecture and owner.                                              |
| S3 bucket name already exists             | Choose another globally unique name.                                                          |
| EC2 website times out                     | Public IP, public route, Internet Gateway, security group, instance status and Nginx.         |
| `InvalidSubnetID.NotFound`                | Subnet and VPC configuration and selected region.                                             |
| Terraform proposes replacement            | Read the `-/+` markers and inspect which attribute forces replacement.                        |
| State lock error                          | Confirm no other Terraform operation is running before investigating a stale lock.            |

Useful diagnostic commands:

```
aws sts get-caller-identity
aws configure list
terraform state list
terraform plan
```

`aws sts get-caller-identity` checks which AWS identity your credentials represent. It does not prove that the identity has all required permissions.

## Lesson 9 — State, drift and safe Terraform habits

Keep these three things separate:

- Configuration: what you want, written in `.tf` files.
- State: Terraform's record of managed resources and their identifiers.
- Actual infrastructure: what exists in AWS right now.

If someone changes a resource manually in AWS, Terraform may detect drift during refresh and planning. Review the plan before applying corrections.

For a local learning project:

- Never commit `terraform.tfstate`, state backups, credentials, or secret-filled `.tfvars` files to Git.
- Keep `.terraform/` out of Git.
- Commit `.tf` files and a `.terraform.lock.hcl` file.
- Remember that marking a value `sensitive` hides it from some output displays; it does not automatically encrypt or remove it from state.

A suitable `.gitignore` is:

```
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfplan
crash.log
```

If you later collaborate with a team, learn remote state storage and state locking before sharing infrastructure management.

## Lesson 10 — Cost control and cleanup

EC2 instances, public IPv4 addresses, EBS storage, and some other AWS features can incur charges. Free-tier eligibility varies by account and current AWS terms. S3 storage and requests can also cost money.

When you finish the lab, destroy the resources you created if you no longer need them:

```
terraform plan -destroy
```

Review the proposed deletions. If correct:

```
terraform destroy
```

Type `yes` after reviewing.

Important: Terraform may fail to delete the S3 bucket if it contains objects or retained object versions. Empty the bucket, including old versions and delete markers where applicable, and retry. Do not delete infrastructure managed by someone else or by another Terraform project.

Verify what remains:

```
terraform state list
```

After successful destruction, the state list should be empty for this project. You can also check the EC2 and S3 consoles to confirm that the lab resources are gone.

## Final Module 3 interview recap

| Interview question               | Short answer                                                                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| What is a VPC?                   | A logically isolated network in AWS.                                                                                                  |
| What makes a subnet public?      | Its route table has a route to an Internet Gateway; internet reachability also requires appropriate IP addressing and security rules. |
| What is a Security Group?        | A stateful virtual firewall for AWS resources.                                                                                        |
| What is an AMI?                  | A template used to launch an EC2 instance.                                                                                            |
| What is user data?               | A startup script used to initialize an EC2 instance.                                                                                  |
| What is EBS?                     | Block storage commonly attached to EC2 instances.                                                                                     |
| What is an IAM role?             | An identity that provides temporary permissions to an AWS service or trusted principal.                                               |
| What is an instance profile?     | The container through which an IAM role is associated with an EC2 instance.                                                           |
| What is an S3 bucket?            | A container for storing objects in Amazon S3.                                                                                         |
| What is a Terraform data source? | A way to read information about an existing object, such as an AMI.                                                                   |
| What is a Terraform resource?    | A configuration block used to create and manage infrastructure.                                                                       |
| Why use outputs?                 | To display useful values such as an instance ID or website URL.                                                                       |

## Your completion checklist

Module 3 practical tracker

0/10

Review existing VPC and networking resources

Add security group

Add EC2, AMI lookup and Nginx user data

Configure encrypted EBS storage

Create S3 bucket with encryption and public access blocking

Configure IAM role and instance profile

Add outputs and validate dependencies

Review plan and apply successfully

Verify website and AWS resources

Destroy lab resources and verify cleanup

One important distinction: reading the code completes the theory lesson, but the hands-on module is complete only after you validate the configuration, review the plan, apply successfully, verify the website, and clean up the AWS resources.
