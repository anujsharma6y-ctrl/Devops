✅ Correct! B — Modules help you reuse and organize Terraform code.

For example, write an EC2 module once and reuse it for `dev`, `test`, and `prod` with different inputs.

Now let's move to the next lesson in our shortened plan.

# Lesson 5 — AWS infrastructure with Terraform

We'll learn how to create the core AWS resources a junior DevOps engineer should understand:

1. VPC — your private AWS network.
2. Subnet — a smaller network inside the VPC.
3. Security group — controls allowed network traffic.
4. EC2 — a virtual server.

VPC

Your AWS network

Subnet

Network segment

Security Group + EC2

Traffic rules + virtual server

## Step 1: Configure the AWS provider

Create a new folder so you don't mix this with your local-file exercises:

```
mkdir -p ~/terraform-aws
cd ~/terraform-aws
nano main.tf
```

Start with this code:

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

This selects the AWS provider and the Mumbai Region.

Before creating any resources: Check whether your AWS credentials are configured and which account they use:

```
aws sts get-caller-identity
```

If the AWS CLI isn't installed or credentials aren't configured, stop here rather than creating anything. Don't paste credentials or secret keys into chat.

## Step 2: Understand the infrastructure code

We will add the VPC, subnet, security group, and EC2 in stages, checking each with `terraform validate` and `terraform plan` before applying. Creating EC2 can incur charges, so we'll use a small instance and clean up afterward.

One quick question before we continue: Have you already configured AWS credentials in your Ubuntu/WSL terminal?
Great. Let's verify your AWS access first, then we'll create the infrastructure step by step.

## Step 1 — Verify AWS credentials

Run this in your Ubuntu/WSL terminal:

```
aws sts get-caller-identity
```

You should see output containing an account ID and an ARN. Don't share the full output publicly.

## Step 2 — Verify Terraform

From your `~/terraform-aws` directory, run:

```
terraform init
terraform fmt
terraform validate
terraform plan
```

At this stage, because we have only configured the AWS provider and haven't defined any resources, Terraform should report no changes.

## Step 3 — Tell me the result

Both AWS identity and Terraform commands worked

AWS identity command failed

Terraform command failed

Continue

Once verified, we'll add the VPC and subnet, then the security group and EC2. We'll review the plan before creating any billable resources.
Great — your AWS credentials and Terraform setup are working. Now we'll create your first AWS network using Terraform.

# Lesson 5 — Step 1: Create a VPC

A VPC (Virtual Private Cloud) is an isolated network in AWS where you place resources such as EC2 instances.

We'll create a VPC in the Mumbai Region (`ap-south-1`) with the IPv4 range `10.0.0.0/16`.

## AWS VPC

CIDR: 10.0.0.0/16

Next: Subnet

We'll create this in the next step.

## 1. Add the VPC resource

Open your existing file:

```
cd ~/terraform-aws
nano main.tf
```

Keep your existing `terraform` and `provider` blocks. Add this resource underneath them:

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

Save with `Ctrl+O`, press `Enter`, then `Ctrl+X`.

What the important settings mean:

- `cidr_block`: the private IPv4 address range for the VPC.
- `enable_dns_support`: enables DNS resolution in the VPC.
- `enable_dns_hostnames`: enables DNS hostnames for eligible instances.
- `tags`: gives the resource a recognizable name.

## 2. Validate and preview

Run these commands in order:

```
terraform fmt
terraform validate
terraform plan
```

Your plan should show 1 resource to add — `aws_vpc.main`. The VPC configuration follows the AWS provider's documented resource syntax.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://registry.terraform.io\&sz=32)

Terraform Registry



## 3. Create the VPC

If validation succeeds and the plan shows only the VPC you intended to create, run:

```
terraform apply
```

Review the plan and type `yes` when prompted.

Then verify the VPC ID:

```
terraform state show aws_vpc.main
```

Look for `id`, which should look like `vpc-...`.

## Your checkpoint

Run the commands and tell me which result you get:

VPC created successfully

Permission / AccessDenied error

Terraform syntax or validation error

Another error

Continue

Next we'll create a subnet inside this VPC, using `aws_vpc.main.id` to connect the resources.
Great! Your VPC is created successfully. Now let's create a subnet inside that VPC.

# Lesson 5 — Step 2: Create a subnet

A subnet is a smaller IP address range inside a VPC. Later, we'll place your EC2 instance inside this subnet.

VPC — `10.0.0.0/16`

Subnet — `10.0.1.0/24`

A smaller range inside the VPC

## 1. Add the subnet resource

Open your existing `main.tf`:

```
cd ~/terraform-aws
nano main.tf
```

Keep your provider and VPC code. Add this resource below the VPC:

```

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "terraform-learning-subnet"
  }
}

```

Understand these important lines:

- `vpc_id = aws_vpc.main.id` — attaches the subnet to the VPC you created.
- `cidr_block = "10.0.1.0/24"` — assigns a smaller IP range that fits inside the VPC.
- `availability_zone` — chooses an Availability Zone in Mumbai.
- `map_public_ip_on_launch = true` — requests public IPv4 addresses for eligible instances launched in this subnet. It does not by itself make the subnet internet-accessible.

## 2. Validate and preview

Save the file (`Ctrl+O`, `Enter`, `Ctrl+X`) and run:

```
terraform fmt
terraform validate
terraform plan
```

Because your VPC already exists in Terraform state, the expected plan is 1 resource to add: the subnet, with no replacement of the VPC.

If the plan looks correct, apply it:

```
terraform apply
```

Type `yes` after reviewing the plan.

## 3. Verify the subnet

```
terraform state show aws_subnet.public
```

Look for `id`, `vpc_id`, and `cidr_block`.

Remember: A subnet configured to assign public IPs is not automatically a working public subnet. Internet access also requires appropriate routing, such as a route to an Internet Gateway. We'll cover networking before launching EC2.

## Your checkpoint

Did the subnet get created successfully?

Subnet created successfully

Permission or AccessDenied error

CIDR or Availability Zone error

Another error

Continue
# Lesson 5 — Complete AWS Terraform lab

We'll finish the remaining AWS infrastructure in one pass: Internet Gateway → route table → security group → EC2 → outputs → verification → cleanup.

You already created the VPC and started creating the subnet. We'll keep using `~/terraform-aws` and your existing Terraform state, so Terraform can track the resources together.

## Architecture you'll build

Internet

Internet Gateway

VPC — `10.0.0.0/16`

Public subnet — `10.0.1.0/24`

Route table: `0.0.0.0/0` → Internet Gateway

Security group

SSH access only from your IP

EC2 virtual server

Security note: We won't open SSH to everyone (`0.0.0.0/0`). We'll restrict it to your public IP. EC2 can incur charges, so the final section includes cleanup.

## 1. Add the remaining resources

Open your existing file:

```
cd ~/terraform-aws
nano main.tf
```

Replace the contents with this complete configuration. It includes your existing VPC and subnet resources, keeping their original Terraform resource names so existing state can be reused.

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

# 1. VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "terraform-learning-vpc"
  }
}

# 2. Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "terraform-learning-subnet"
  }
}

# 3. Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "terraform-learning-igw"
  }
}

# 4. Route table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "terraform-learning-public-rt"
  }
}

# 5. Associate route table with subnet
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# 6. Security group
variable "ssh_cidr" {
  description = "Your public IPv4 address followed by /32"
  type        = string
}

resource "aws_security_group" "web" {
  name        = "terraform-learning-sg"
  description = "SSH access restricted to my IP"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from my IP only"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_cidr]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "terraform-learning-sg"
  }
}

# 7. Find a recent Amazon Linux 2023 AMI
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
    name   = "state"
    values = ["available"]
  }
}

# 8. EC2 instance
variable "key_name" {
  description = "Existing EC2 key pair name in ap-south-1"
  type        = string
}

resource "aws_instance" "web" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t3.micro"
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.web.id]
  key_name                    = var.key_name
  associate_public_ip_address = true

  tags = {
    Name = "terraform-learning-ec2"
  }
}

# 9. Outputs
output "vpc_id" {
  value = aws_vpc.main.id
}

output "subnet_id" {
  value = aws_subnet.public.id
}

output "ec2_public_ip" {
  value = aws_instance.web.public_ip
}

output "ec2_instance_id" {
  value = aws_instance.web.id
}

```

Save with `Ctrl+O`, press `Enter`, then `Ctrl+X`.

## 2. Configure your IP address and key pair

First, find your public IPv4 address:

```
curl -4 https://checkip.amazonaws.com
```

Suppose it returns `203.0.113.25`. Your SSH CIDR would be `203.0.113.25/32`. That address is only an example; use your actual result.

Next, list available EC2 key pairs in Mumbai:

```
aws ec2 describe-key-pairs \
  --region ap-south-1 \
  --query 'KeyPairs[*].KeyName' \
  --output table
```

Choose an existing key pair whose private `.pem` file you possess. If the list is empty, create a key pair in the AWS console before continuing.

Create a Terraform variables file:

```
nano terraform.tfvars
```

Enter your actual values:

```
ssh_cidr = "YOUR_PUBLIC_IP/32"
key_name = "YOUR_EXISTING_KEY_PAIR_NAME"
```

Replace both placeholders. Don't put your private key or AWS credentials in this file.

## 3. Validate the complete configuration

Run:

```
terraform fmt
terraform validate
terraform plan
```

Review the plan carefully.

- If your VPC and subnet already exist in Terraform state, they should be reused rather than recreated.
- You should see the remaining networking resources, security group and EC2 proposed for creation.
- If Terraform proposes replacing your existing VPC or subnet, stop and inspect the plan before applying.

## 4. Create everything

If the plan is correct, run:

```
terraform apply
```

Review the proposed changes and type `yes` when prompted.

When it finishes, get the outputs:

```
terraform output
```

Check the EC2 instance state:

```
aws ec2 describe-instances \
  --region ap-south-1 \
  --filters "Name=tag:Name,Values=terraform-learning-ec2" \
  --query 'Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]' \
  --output table
```

The instance should eventually show `running` and a public IP.

## 5. What you've built

| Resource                | Purpose                                          |
| ----------------------- | ------------------------------------------------ |
| VPC                     | Isolated AWS network                             |
| Subnet                  | IP range inside the VPC                          |
| Internet Gateway        | Connects the VPC to the internet through routing |
| Route table             | Routes internet-bound traffic to the gateway     |
| Route table association | Applies that route table to the subnet           |
| Security group          | Allows SSH only from your specified IP           |
| AMI data source         | Finds an available Amazon Linux 2023 image       |
| EC2                     | Runs your virtual server                         |
| Outputs                 | Displays useful resource IDs and the public IP   |

Important: A public IP and route don't guarantee SSH access; your IP must be correct, the instance must be running, and network access must be allowed. Don't open port 22 to everyone to troubleshoot.

## 6. Cleanup after the lab

EC2 may incur charges, and a public IPv4 address can also incur charges. When you're finished, from the same project directory run:

```
terraform destroy
```

Review the destruction plan and confirm only when you're ready to delete the lab resources. This configuration manages the VPC and subnet too, so those will also be destroyed.

Your next checkpoint: Complete `terraform validate` and `terraform plan` first. If the plan is correct, apply it. If an error appears, share the error text without any credentials or private keys.
