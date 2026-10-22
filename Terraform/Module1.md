# Terraform Module 1 — Fundamentals & Workflow

Level: Beginner · Track: Junior DevOps · Language: Simple English + Hinglish

In this module, we'll learn what Terraform does, how it works, and how to use its basic workflow. By the end, you should be able to explain Terraform in an interview and run a basic Terraform project yourself.

## Module 1 checklist

Your module progress

## 0/9

What Terraform is and why we use it

Infrastructure as Code (IaC)

Terraform vs Ansible

Declarative approach and idempotency

Providers, resources, data sources and state

Terraform installation and AWS CLI basics

Terraform project files

Core commands and workflow

Dependencies and resource creation order

Use this checklist to track your progress through the lesson.

## Lesson 1. What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool. It lets you create, modify and manage infrastructure using configuration files instead of creating everything manually.

Infrastructure includes things like:

- EC2 virtual machines
- VPCs, subnets and security groups
- S3 buckets
- Load balancers and databases

### Example: without Terraform vs with Terraform

Without Terraform

1. Open AWS Console.
2. Click Create instance.
3. Select an AMI and instance type.
4. Configure networking and security.
5. Repeat manually for other environments.

With Terraform

1. Write the infrastructure configuration.
2. Preview the proposed changes.
3. Apply the configuration.
4. Terraform creates the requested infrastructure.

For example, this configuration describes an AWS EC2 instance:

```
resource "aws_instance" "web" {
  ami           = "ami-REPLACE_WITH_VALID_AMI"
  instance_type = "t3.micro"
}
```

This is an illustrative example, not ready to run: you must choose a valid AMI for your region and configure the AWS provider and credentials.

Hinglish: Terraform mein hum AWS Console par baar-baar click karne ke bajaye code likhte hain. Terraform us code ke according infrastructure create aur manage karta hai.

Interview answer: “Terraform is an Infrastructure as Code tool used to provision and manage cloud infrastructure through declarative configuration files.”

## Lesson 2. What is Infrastructure as Code (IaC)?

IaC means managing infrastructure through code or configuration files.

Think of it like this:

Terraform configuration

Describe the infrastructure you want

Terraform plan

Preview proposed changes

Terraform apply

Create or modify resources

Infrastructure

For example, an EC2 instance and its network

### Why do DevOps engineers use IaC?

- Repeatability: create similar infrastructure consistently.
- Version control: track changes using Git.
- Automation: run infrastructure changes through CI/CD pipelines.
- Review: inspect a plan before applying changes.
- Consistency: reduce manual configuration mistakes.
- Recovery: recreate infrastructure from maintained configuration, provided dependencies and required data are available.

IaC does not automatically make infrastructure secure or error-free. Bad configuration can still create insecure resources or cause downtime.

Interview answer: “Infrastructure as Code means defining and managing infrastructure through configuration files so that it can be automated, reviewed and reproduced.”

## Lesson 3. Terraform vs Ansible

This is one of the most important concepts for junior DevOps interviews.

| Terraform                                 | Ansible                                                                                              |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Mainly provisions infrastructure          | Mainly configures servers and deploys software                                                       |
| Creates EC2, VPC, S3, etc.                | Installs Nginx, packages and application files                                                       |
| Uses HCL                                  | Uses YAML playbooks                                                                                  |
| Tracks managed infrastructure using state | Typically connects to hosts and executes tasks; it does not use Terraform-style infrastructure state |
| Commonly communicates with cloud APIs     | Commonly uses SSH for Linux servers                                                                  |

### Real-world example

Suppose your company needs an EC2 server running Nginx.

Terraform

Creates the VPC, security group and EC2 instance

Ansible

Connects to EC2, installs Nginx and configures the service

Working application

Infrastructure exists and the software is configured

Hinglish: Terraform server banata hai; Ansible us server ke andar software install aur configure karta hai. This is a common pattern, although both tools can perform some overlapping tasks.

Interview answer: “Terraform is primarily used for infrastructure provisioning, whereas Ansible is primarily used for configuration management and application deployment.”

## Lesson 4. Declarative approach and idempotency

### A. Declarative vs imperative

Declarative: describe the desired result.

```
resource "local_file" "example" {
  filename = "example.txt"
  content  = "Hello DevOps"
}
```

This declares that a file should exist with the specified content.

Imperative: specify the steps to execute.

```
mkdir -p example
echo "Hello DevOps" > example/example.txt
```

This command explicitly tells the shell what actions to perform.

Terraform is primarily declarative. You describe the desired infrastructure; Terraform calculates the changes needed to reach that state.

### B. What is idempotency?

Idempotency means that repeating an operation produces the same intended end state rather than repeatedly creating unwanted changes.

Example:

1. Terraform creates a file with `Hello DevOps`.
2. You run `terraform apply` again without changing the configuration.
3. Terraform should report no changes if the file still matches the desired state.

Important: idempotency does not mean that every command is harmless to repeat. A shell command that appends text to a file can duplicate content on every run.

Interview answer: “Idempotency means repeatedly applying the same desired configuration does not create additional unnecessary changes.”

## Lesson 5. Five Terraform building blocks

| Term          | Simple meaning                                               | Example                 |
| ------------- | ------------------------------------------------------------ | ----------------------- |
| Provider      | Connects Terraform to a platform or API                      | AWS provider            |
| Resource      | Infrastructure Terraform manages                             | EC2 instance            |
| Data source   | Reads information about an existing object                   | Look up an existing VPC |
| Configuration | Describes the desired setup                                  | `.tf` files             |
| State         | Records Terraform's knowledge and mapping of managed objects | `terraform.tfstate`     |

### How they fit together

- You write configuration.
- The provider communicates with AWS or another platform.
- Resources describe what Terraform should manage.
- Data sources read information Terraform needs.
- State helps Terraform understand previously managed objects and determine what needs to change.

State is important because Terraform needs to associate configuration with real infrastructure. It can contain sensitive values, so don't commit state files to a public Git repository.

## Lesson 6. Installation and basic setup

Since you use Windows with Ubuntu/WSL, you can learn Terraform inside Ubuntu. You do not need to install it separately on Windows if you will run it entirely inside WSL.

Official installation guide: [Install Terraform](https://developer.hashicorp.com/terraform/install?utm_source=chatgpt.com)

In your Ubuntu terminal, check whether Terraform is already installed:

```
terraform -version
```

If the command returns a version, installation is available in that terminal. If it says `command not found`, follow the official installation instructions for Ubuntu.

Check AWS CLI separately:

```
aws --version
```

Terraform can also manage local resources without AWS credentials, which is useful for your first lab.

Important: don't put AWS access keys directly into Terraform files or commit them to Git.

## Lesson 7. Understand the Terraform project files

A basic Terraform project might look like this:

```
terraform-lab/
├── main.tf
├── variables.tf
├── outputs.tf
├── .terraform.lock.hcl
├── .terraform/
└── terraform.tfstate
```

| File/folder           | Purpose                                           |
| --------------------- | ------------------------------------------------- |
| `main.tf`             | Commonly holds resource definitions               |
| `variables.tf`        | Defines input variables                           |
| `outputs.tf`          | Defines values to display after operations        |
| `.terraform.lock.hcl` | Records selected provider versions and checksums  |
| `.terraform/`         | Contains downloaded provider and working metadata |
| `terraform.tfstate`   | Local state file, when using the local backend    |

These filenames are conventions, not strict requirements. Terraform reads configuration files ending in `.tf` from the working directory.

For a beginner, start with just `main.tf`. Split the code into more files when the project grows.

## Lesson 8. The six core Terraform commands

Learn this sequence first.

### 1

terraform init

Initialises the working directory and downloads required providers.

&#x20;Copy

### 2

terraform fmt

Formats Terraform configuration consistently.

&#x20;Copy

### 3

terraform validate

Checks configuration syntax and internal consistency.

&#x20;Copy

### 4

terraform plan

Previews proposed changes without applying them.

&#x20;Copy

### 5

terraform apply

Executes the planned infrastructure changes after confirmation, by default.

&#x20;Copy

### 6

terraform destroy

Plans and destroys resources managed by the current configuration and state.

&#x20;Copy

Typical workflow:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Run `terraform destroy` only when you intend to remove the managed resources. Always review the plan, particularly before deleting or replacing resources.

### Two common beginner mistakes

- `terraform init` does not create your EC2 instance. It prepares the working directory.
- `terraform plan` previews changes. It does not normally apply them.

## Lesson 9. Resource dependencies

Suppose Terraform must create a VPC before creating a subnet that belongs to it.

```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}
```

Terraform sees `aws_vpc.main.id` referenced by the subnet and infers the dependency automatically.

VPC

Subnet

Terraform creates the VPC first because the subnet needs its ID.

You can also use `depends_on` for dependencies Terraform cannot infer from direct references, but don't add it everywhere. Unnecessary dependencies can restrict Terraform's ability to work efficiently.

Interview answer: “Terraform builds a dependency graph and determines the order of operations. References between resources usually create implicit dependencies.”

## Practical lab — Your first Terraform project

We'll create a local text file, so this lab does not need AWS credentials, a paid cloud resource or administrator-level AWS permissions.

### Step 1: Create a project folder

Run these commands in your Ubuntu/WSL terminal:

```
mkdir -p ~/terraform-lab
cd ~/terraform-lab
```

Confirm your location:

```
pwd
```

You should see a path similar to:

```
/home/your-username/terraform-lab
```

### Step 2: Create `main.tf`

Open the file:

```
nano main.tf
```

Paste this configuration:

```
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

resource "local_file" "hello" {
  filename = "${path.module}/hello.txt"
  content  = "Hello from Terraform!"
}

output "file_path" {
  value = local_file.hello.filename
}
```

Save in Nano:

1. Press `Ctrl + O`.
2. Press `Enter`.
3. Press `Ctrl + X`.

If you use Nano and the save screen confuses you, remember that `Ctrl + O` saves the file; `Ctrl + X` exits the editor.

### Step 3: Initialise the project

```
terraform init
```

Terraform should download the Local provider and initialise the directory. If this fails, check your internet connection and the error message.

### Step 4: Format and validate

```
terraform fmt
terraform validate
```

Expected validation result:

```
Success! The configuration is valid.
```

### Step 5: Preview the changes

```
terraform plan
```

Review the plan. You should see Terraform propose creating a local file resource.

### Step 6: Apply the configuration

```
terraform apply
```

Read the proposed actions. Type `yes` when prompted to confirm.

Check the file:

```
cat hello.txt
```

Expected output:

```
Hello from Terraform!
```

### Step 7: Inspect the result

```
terraform output
terraform state list
```

You should see the output path and the resource address:

```
local_file.hello
```

Now run:

```
terraform plan
```

If you haven't changed the configuration or the file, Terraform should normally report that no changes are needed.

### Step 8: Clean up

```
terraform destroy
```

Review the plan and confirm with `yes`. Terraform should remove the managed file.

The local state and other working files may remain. That is expected; destroying the managed resource does not automatically delete the entire project directory.

Lab success criteria: you can explain what `init`, `plan`, `apply`, `output`, `state list` and `destroy` did, rather than simply copying the commands.

## Module 1 test — Check your understanding

Answer these 8 questions without looking back at the notes. Target: 7/8 or higher. We'll review every answer and explain why the other options are incorrect before moving to Module 2.

1\. What is Terraform primarily used for?

A. Writing application code

B. Provisioning and managing infrastructure as code

C. Monitoring application logs

D. Building Docker images

2\. Which command initialises a Terraform project and downloads required providers?

A. terraform apply

B. terraform validate

C. terraform init

D. terraform destroy

3\. What does terraform plan do?

A. Previews proposed changes

B. Immediately deletes all resources

C. Installs the AWS CLI

D. Always creates resources

4\. What is a Terraform provider?

A. A file that stores passwords

B. A plugin that lets Terraform interact with a platform or API

C. A resource created by every configuration

D. A replacement for Terraform state

5\. Which statement about Ansible is most accurate?

A. It is only used to create VPCs

B. It replaces all Terraform functionality

C. It is commonly used to configure servers and deploy software

D. It is an AWS-only tool

6\. Terraform configuration is unchanged and the real resource matches it. What should a new plan normally show?

A. A new resource every time

B. No changes needed

C. Destroy everything

D. An automatic provider upgrade

7\. Why does Terraform use state?

A. To store application logs

B. To help track managed objects and map configuration to infrastructure

C. To replace all .tf files

D. To install Linux packages

8\. A subnet references aws_vpc.main.id. What does Terraform normally infer?

A. The subnet must be created before the VPC

B. There is no dependency

C. The VPC must be created before the subnet

D. Both resources are automatically deleted

0/8 answered

Submit answers and review

Don't worry if you get some wrong. The aim is to understand the reasoning, not memorise answers.
