# Terraform Module 4 — State, Modules, Workspaces & Reusable Code

This module covers the complete junior DevOps–relevant Terraform state and reusability syllabus in one lesson. We’ll cover concepts, practical commands, code examples, common errors, and interview questions.

## Complete Module 4 checklist

Progress checklist

0/9

1\. Terraform state

State file, purpose, resource mapping, state commands

2\. Configuration vs state vs real infrastructure

How Terraform compares desired and actual infrastructure

3\. Drift detection

Manual AWS changes and how Terraform detects them

4\. State operations

list, show, mv, rm, pull and import

5\. Remote state

S3 backend, collaboration, locking and security

6\. Modules

Root modules, child modules, inputs, outputs and reuse

7\. Environments

Separate dev/prod configuration and workspaces

8\. Import and moved blocks

Adopting existing resources and renaming safely

9\. Recovery and best practices

Backups, state protection, common errors

## 1. What is Terraform state?

Terraform state is a file or remote record that maps Terraform configuration to real infrastructure.

For example, your configuration contains:

```
resource "aws_s3_bucket" "learning" {
  bucket = "my-unique-learning-bucket-123"
}
```

Terraform needs to remember which real AWS bucket belongs to `aws_s3_bucket.learning`. State maintains that association and other information needed to plan changes.

By default, Terraform uses a local file called:

```
terraform.tfstate
```

### Why is state needed?

- Track resources Terraform manages.
- Remember real resource IDs.
- Calculate changes between runs.
- Understand dependencies and resource relationships.
- Support updates and deletions.

Important: State is not your configuration, and it is not a complete backup of your AWS infrastructure.

## 2. Configuration vs state vs actual infrastructure

Configuration (`.tf`)

What you want

Terraform plan / apply and refresh

State (`terraform.tfstate`)

What Terraform records about managed resources

Compared with actual infrastructure

AWS infrastructure

What actually exists

Example:

- Configuration says an EC2 instance should be `t3.micro`.
- State records the instance Terraform manages.
- AWS currently has that instance running as `t3.small`, because someone changed it manually.

A Terraform plan may propose changing the instance back to match the configuration. Whether that requires replacement depends on the attribute and provider behavior.

## 3. State commands you should know

Run these inside your Terraform project.

| Command                        | Meaning                                                          |
| ------------------------------ | ---------------------------------------------------------------- |
| `terraform state list`         | List resources recorded in state.                                |
| `terraform state show ADDRESS` | Show details of one resource.                                    |
| `terraform state pull`         | Print the current state snapshot.                                |
| `terraform state mv OLD NEW`   | Change a resource's address in state.                            |
| `terraform state rm ADDRESS`   | Remove a resource from state without deleting the real resource. |
| `terraform show`               | Display the current state or saved plan.                         |
| `terraform plan`               | Calculate proposed changes.                                      |

Examples:

```
terraform state list
terraform state show aws_instance.web
terraform show
```

Use the exact resource address shown by `terraform state list`.

### Dangerous distinction

`terraform state rm` does not delete the EC2 instance from AWS. It only removes Terraform's tracking of it. A later plan could propose creating another instance.

`terraform destroy` is different: it plans to delete the managed infrastructure.

Never edit `terraform.tfstate` manually as a routine workflow.

## 4. Drift detection

Drift means actual infrastructure no longer matches the configuration Terraform expects.

Example:

1. Terraform creates an EC2 instance with a particular instance type.
2. Someone changes its configuration directly in AWS.
3. You run `terraform plan`.
4. Terraform detects the difference if the changed attribute is observable.
5. The plan proposes a correction, replacement, or another appropriate action.

Commands:

```
terraform plan
terraform plan -refresh-only
```

`terraform plan -refresh-only` previews state updates based on observed infrastructure; it does not apply them automatically.

Use a normal plan to see proposed infrastructure changes. Don't blindly apply changes you don't understand, especially when a resource may be replaced or deleted.

## 5. Remote state using S3

Local state works for a solo lab, but sharing it across a team is risky. Two people could run Terraform against separate local states and create conflicting changes.

A remote backend allows authorized team members to use a shared state location.

### Basic S3 backend configuration

Create an S3 bucket specifically for state storage, with an appropriate unique name, restricted access, encryption and versioning. Then add a backend block to the root configuration:

```
terraform {
  backend "s3" {
    bucket       = "replace-with-your-state-bucket"
    key          = "learning/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

This is a configuration example: replace the bucket name with a real bucket you control before initializing the backend.

Then run:

```
terraform init
```

Terraform will initialize the S3 backend. If migrating from local state, review the migration prompt and preserve the existing state.

State backend security checklist:

- Restrict bucket access using least-privilege IAM policies.
- Enable S3 versioning to help recover earlier state versions.
- Use encryption and appropriate bucket security controls.
- Do not store AWS access keys in Terraform files.
- Use locking to reduce simultaneous state modifications.
- Back up and test recovery procedures.

`use_lockfile = true` enables native S3 state locking in supported Terraform versions. Don't assume DynamoDB locking is required for every new S3 backend setup.

Interview answer: Remote state centralizes Terraform state for collaboration; locking helps prevent concurrent operations from corrupting or conflicting over the same state.

## 6. Terraform modules

A module is a reusable group of Terraform configuration files.

For example, instead of writing the same VPC configuration for every environment, you can create one VPC module and call it from different root configurations.

### Typical project structure

```
terraform-lab/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── modules/
    └── s3-bucket/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Child module: `modules/s3-bucket/variables.tf`

```
variable "bucket_name" {
  type = string
}
```

### Child module: `modules/s3-bucket/main.tf`

```
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
}
```

### Child module: `modules/s3-bucket/outputs.tf`

```
output "bucket_id" {
  value = aws_s3_bucket.this.id
}
```

### Call the module from your root `main.tf`

```
module "app_bucket" {
  source      = "./modules/s3-bucket"
  bucket_name = var.bucket_name
}

output "app_bucket_id" {
  value = module.app_bucket.bucket_id
}
```

This assumes you have already declared `var.bucket_name` in the root module and supplied a valid globally unique bucket name.

Run:

```
terraform fmt -recursive
terraform init
terraform validate
terraform plan
```

Modules help with reuse and consistency, but they don't automatically guarantee security or quality. Validate module inputs, outputs, permissions and resource behavior.

## 7. Root modules vs child modules

| Root module                           | Child module                                |
| ------------------------------------- | ------------------------------------------- |
| The directory where you run Terraform | A module called by another module           |
| Often defines the environment         | Packages reusable infrastructure            |
| Supplies inputs to child modules      | Accepts variables and returns outputs       |
| Uses backend configuration            | Normally does not configure its own backend |

A module can be local, such as `./modules/network`, or sourced from a registry or supported Git source.

## 8. Variables, outputs and locals inside modules

- Variables: Inputs passed into a module.
- Outputs: Values returned by a module.
- Locals: Named expressions used internally.

Example:

```
variable "environment" {
  type = string
}

locals {
  resource_name = "web-${var.environment}"
}

output "resource_name" {
  value = local.resource_name
}
```

When the root module passes `environment = "dev"`, the output becomes `web-dev`.

## 9. Dev and production environments

For a beginner project, separate root configurations are often easy to understand:

```
environments/
├── dev/
│   ├── main.tf
│   └── terraform.tfvars
└── prod/
    ├── main.tf
    └── terraform.tfvars
```

Both environments can reuse the same child modules but use separate state locations and different input values.

Production should use suitable isolation, access controls, review and deployment approvals. Do not assume that using separate folders alone isolates the infrastructure.

## 10. Terraform workspaces

Workspaces let one configuration use multiple state instances.

Useful commands:

```
terraform workspace list
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
terraform workspace show
```

Workspaces can be useful for simple variations, but they are not a complete environment-isolation or access-control system. For production environments requiring strong separation, separate root configurations and separate state/backend arrangements are often easier to secure and reason about.

## 11. Import existing infrastructure

Suppose a resource already exists in AWS and you want Terraform to manage it rather than create a duplicate.

First, write a matching resource block. Then you can use an import block:

```
import {
  to = aws_s3_bucket.learning
  id = "your-existing-bucket-name"
}
```

The target resource address must exist in your configuration. Then run:

```
terraform plan
terraform apply
```

Review the plan carefully. Importing brings the existing resource under Terraform's management; it does not automatically generate a perfect configuration for every existing setting.

A CLI alternative is:

```
terraform import aws_s3_bucket.learning your-existing-bucket-name
```

For either approach, make sure the resource configuration matches the real resource before applying future changes.

## 12. Renaming a resource safely with `moved`

Suppose you rename a resource in your configuration:

```
resource "aws_s3_bucket" "app" {
  bucket = var.bucket_name
}
```

Previously, it was called `aws_s3_bucket.learning`.

Without migration guidance, Terraform may interpret this as removing one resource and creating another. A `moved` block tells Terraform the resource's address changed:

```
moved {
  from = aws_s3_bucket.learning
  to   = aws_s3_bucket.app
}
```

Run `terraform plan` and confirm it shows the address migration rather than unnecessary resource destruction and recreation.

## 13. State recovery and common errors

| Problem                                    | Safe response                                                                                            |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| State file missing                         | Stop and investigate backups, remote state and the current infrastructure before applying.               |
| State lock error                           | Check whether another Terraform operation is active before considering lock recovery.                    |
| Resource exists but state doesn't track it | Inspect configuration and actual resource, then consider importing.                                      |
| Manual AWS changes                         | Run a plan and understand the proposed reconciliation.                                                   |
| Wrong resource address                     | Inspect state and use `terraform state mv` or a `moved` block where appropriate.                         |
| State contains secrets                     | Restrict access, protect backups and avoid exposing state contents.                                      |
| Accidental resource replacement            | Stop before applying; inspect which attribute forces replacement and revise the configuration if needed. |

Do not run `terraform force-unlock` or remove state records just to make an error disappear. These operations can affect other Terraform users or leave resources unmanaged.

## Practical lab — what to do on your existing project

You don't need to recreate your AWS infrastructure for this module.

1. Inspect resources using `terraform state list`.
2. Inspect a resource using `terraform state show`.
3. Run `terraform plan` and explain any proposed changes.
4. Create a small local child module that outputs a string, so you can practise module inputs and outputs without creating AWS resources.
5. Review how an S3 backend and `moved` block work using the examples above.
6. Do not migrate your real state or import production resources just for practice.

## Final interview revision

| Question                           | Short answer                                                                             |
| ---------------------------------- | ---------------------------------------------------------------------------------------- |
| What is Terraform state?           | It maps Terraform configuration to managed infrastructure.                               |
| What is drift?                     | A difference between expected configuration and actual infrastructure.                   |
| Why use remote state?              | To share state securely and consistently between authorized team members.                |
| What is state locking?             | A mechanism to reduce conflicting concurrent state operations.                           |
| What is a module?                  | Reusable Terraform configuration packaged together.                                      |
| What is a workspace?               | A separate state instance associated with the same configuration.                        |
| What does `terraform state rm` do? | Removes a resource from state without deleting the actual resource.                      |
| What does `terraform import` do?   | Associates an existing resource with a Terraform resource address.                       |
| What is a `moved` block?           | It tells Terraform how a resource address changed, helping avoid unnecessary recreation. |
| Why protect state?                 | It may contain sensitive infrastructure details and secret values.                       |

Module 4 is covered. For a junior DevOps role, focus most on understanding state, reading plans, using modules, managing remote state safely, and knowing when to import instead of recreating resources.
