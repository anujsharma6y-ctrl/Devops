# Terraform Module 5 — Git, CI/CD, Security & Production Workflow

This is the final core Terraform module for your junior DevOps preparation. We’ll cover the theory and hands-on workflow together, using simple commands and a practical project.

## Complete checklist

Module checklist

0/9

Git and Terraform project structure

.gitignore and protecting state/secrets

Git workflow: branch, commit and pull request

Terraform fmt, validate and plan in CI

Review and approve changes before apply

Jenkins and GitHub Actions integration

AWS credentials, IAM least privilege and OIDC

Failed deployments, replacement and rollback

Final end-to-end practical

## 1. Git and Terraform project structure

Git tracks changes to your Terraform code. Terraform state tracks managed infrastructure. They are not the same thing.

A typical project:

```
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── .gitignore
└── .terraform.lock.hcl
```

- `.tf` files: infrastructure configuration.
- `.terraform.lock.hcl`: records selected provider versions and checksums.
- `.terraform/`: downloaded providers and other working data.
- `terraform.tfstate`: local state, which may contain sensitive data.
- `terraform.tfvars`: variable values, which may contain secrets.

## 2. Protect your files with `.gitignore`

Inside your project, create or edit `.gitignore`:

```
cd ~/terraform-state-module-lab
nano .gitignore
```

Add:

```
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
*.tfplan
crash.log
```

Save using `Ctrl+O`, `Enter`, `Ctrl+X`.

Why ignore these files?

- State files can contain credentials or sensitive infrastructure details.
- `.tfvars` files may contain secrets.
- `.terraform/` contains local working data.
- Saved plan files can contain sensitive values too.

Keep `.terraform.lock.hcl` in Git. It helps your team consistently use the selected provider versions.

If you need to share non-secret variable values, create a file such as `terraform.tfvars.example` with placeholder values.

Important: `.gitignore` does not remove files that Git already tracks. If state or secrets were committed previously, you must remove them from tracking and rotate any exposed credentials.

## 3. Git workflow — branch, commit and pull request

Check Git first:

```
git --version
```

Inside your Terraform project:

```
cd ~/terraform-state-module-lab
git status
```

If this project is not already a Git repository, initialize it:

```
git init
git add .gitignore main.tf
git add modules/
git status
```

Review the files listed before committing. Do not add state files, secret-filled `.tfvars`, credentials, or generated plan files.

Then commit:

```
git commit -m "Add reusable Terraform module"
```

If Git asks you to configure your identity, configure your Git author name and email first.

### Typical team workflow

Create feature branch

Edit Terraform files

Run fmt, validate and plan

Open pull request and review plan

Approval → controlled apply

Commands commonly used:

```
git switch -c feature/terraform-network
git add main.tf
git commit -m "Update Terraform configuration"
git push -u origin feature/terraform-network
```

The push command requires a configured Git remote, such as your GitHub repository.

## 4. Terraform CI — automate quality checks

CI (Continuous Integration) automatically checks changes when code is pushed or a pull request is opened.

The initial automated checks are:

| Command                           | Purpose                                                                                                                    |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `terraform fmt -check -recursive` | Check formatting                                                                                                           |
| `terraform init -backend=false`   | Initialize providers without initializing the configured backend                                                           |
| `terraform validate`              | Check configuration syntax and internal consistency                                                                        |
| `terraform plan`                  | Calculate proposed infrastructure changes; needs appropriate backend access and provider credentials in many real projects |

`validate` does not prove that your AWS credentials work, that your IAM permissions are sufficient, or that a deployment will succeed.

### Hands-on: GitHub Actions

Create this workflow in your repository:

```
mkdir -p .github/workflows
nano .github/workflows/terraform-checks.yml
```

Paste:

```
name: Terraform Checks

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Check formatting
        run: terraform fmt -check -recursive

      - name: Initialize providers
        run: terraform init -backend=false

      - name: Validate configuration
        run: terraform validate
```

This workflow runs basic checks. It intentionally does not apply infrastructure or access AWS credentials. For your current practice project, make sure the workflow runs from the directory containing the root Terraform configuration.

Save and commit:

```
git add .github/workflows/terraform-checks.yml
git commit -m "Add Terraform CI checks"
```

Once pushed to a connected GitHub repository, inspect the Actions tab to see the workflow result.

Note: the example uses versioned third-party actions; in production, pin actions to reviewed commit SHAs according to your organization's security policy.

## 5. Terraform plan review and approval

In a real DevOps workflow, a successful validation is not permission to change production.

A safer process is:

1. Run `terraform plan`.
2. Review additions, updates, replacements and deletions.
3. Ask a reviewer to approve the plan.
4. Apply the reviewed changes using controlled credentials.
5. Verify the deployment and monitor the application.

Plan symbols to recognize:

- `+` — create.
- `~` — update in place.
- `-` — destroy.
- `-/+` — replace a resource, generally destroy and recreate.
- `+/-` — create before destroying the old resource when the provider and lifecycle behavior allow it.

A plan may become stale if the infrastructure or state changes before apply. For important environments, generate and review a saved plan, protect it as a potentially sensitive artifact, and apply that exact plan only while it remains valid.

Never automatically apply every pull request to production without proper safeguards.

## 6. Jenkins integration

Jenkins can run the same Terraform commands as GitHub Actions. A Jenkins pipeline typically performs checkout, formatting, initialization, validation, planning, approval and apply.

Create a `Jenkinsfile` in the repository root:

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Format Check') {
            steps {
                sh 'terraform fmt -check -recursive'
            }
        }

        stage('Initialize') {
            steps {
                sh 'terraform init -backend=false'
            }
        }

        stage('Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Plan') {
            steps {
                sh 'terraform plan'
            }
        }
    }
}
```

This is a validation-and-plan pipeline, not a production deployment pipeline. It assumes Terraform is installed on the Jenkins agent and that the checked-out project is the root configuration.

If you use an S3 backend, you'll need an appropriate backend initialization and secure AWS authentication rather than `-backend=false`.

### Jenkins concepts for interviews

- Controller: Manages jobs and coordinates execution.
- Agent: Runs pipeline steps.
- Pipeline: Defines the automated workflow in a `Jenkinsfile`.
- Credentials store: Securely stores credentials Jenkins needs.
- Webhook: Triggers a job when a repository event occurs.

For Terraform, use an agent with a known Terraform version, restricted AWS permissions, and access only to the necessary state and infrastructure.

## 7. AWS credentials and security

### Never hardcode credentials

Do not write AWS access keys into Terraform files:

```
# DON'T DO THIS
provider "aws" {
  access_key = "example-access-key"
  secret_key = "example-secret-key"
  region     = "ap-south-1"
}
```

Prefer supported credential chains, such as an AWS CLI profile for local development or an IAM role for an AWS-hosted runner.

For example, your local AWS CLI identity can be checked with:

```
aws sts get-caller-identity
```

That confirms which identity is being used, but not whether it has permission to perform every planned operation.

### Least privilege

Give the CI identity only the permissions required for its job.

For example:

- Formatting and validation may need no AWS credentials.
- Planning with a remote backend needs state access and permission to read relevant infrastructure.
- Applying requires the permissions needed to change the resources in scope.

Do not solve access-denied errors by automatically granting `AdministratorAccess`.

### What is OIDC?

OpenID Connect (OIDC) lets a supported CI platform obtain temporary AWS credentials by exchanging a trusted identity token, instead of storing long-lived AWS access keys.

For GitHub Actions, the general setup is:

1. Configure an AWS OIDC identity provider.
2. Create an IAM role trusted by that provider.
3. Restrict the role trust policy to the intended repository and branch or deployment environment.
4. Grant the role only the permissions the workflow needs.
5. Configure the workflow to request an OIDC token and assume that role.

A workflow needs permission such as:

```
permissions:
  contents: read
  id-token: write
```

This permission alone does not grant AWS access. The AWS trust relationship and role policy must also be configured correctly.

Interview answer: OIDC reduces dependence on long-lived secrets by allowing trusted automation to obtain temporary credentials.

## 8. Failed deployments, replacement and recovery

Imagine a plan wants to replace an EC2 instance.

Don't apply immediately. Ask:

1. Which attribute changed?
2. Does the provider require replacement for that attribute?
3. Will the replacement cause downtime?
4. Is the instance storing data that needs to be preserved?
5. Is the plan operating on the intended state and AWS account?

### If `terraform apply` fails

- Read the exact error.
- Check the AWS permissions, quotas, region and resource dependencies.
- Run `terraform plan` again after resolving the problem.
- Inspect actual infrastructure and state before retrying.
- Avoid manually deleting state or forcing state changes just to make the error disappear.

Terraform can successfully create some resources before another resource fails. A failed apply does not guarantee that nothing was created.

### What about rollback?

Terraform is not a universal one-click rollback system. Recovery usually means fixing the configuration, restoring required data or resources, and applying a reviewed plan. State recovery may be necessary if the state itself is damaged.

For applications, deployment strategies such as rolling deployments or blue/green deployments may help reduce downtime, but they require an appropriate architecture.

## 9. Final hands-on lab — Run the complete workflow

Use your practice project:

```
cd ~/terraform-state-module-lab
```

### Step 1: Check your files

```
git status
find . -maxdepth 3 -type f
```

Make sure `.gitignore`, your Terraform files, module files and workflow are present. If Git is not initialized yet, initialize it as shown earlier.

### Step 2: Run the local quality checks

```
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
terraform plan
```

Because this lab uses the `local` provider, you do not need AWS credentials to run it. The plan may show local-file resources being created if you previously destroyed them.

### Step 3: Review the Git changes

```
git status
git diff
```

Confirm that no state files, credentials, or secret-filled variable files are being committed.

If the project is already a Git repository, stage the intended files and commit them:

```
git add .gitignore main.tf modules/ .github/workflows/terraform-checks.yml
git status
git commit -m "Add Terraform module and CI checks"
```

Only run the last command if those paths exist and you have reviewed the staged changes. The Jenkinsfile can also be committed if you created it.

### Step 4: Test your workflow

If you have a GitHub repository connected, push your branch and check the Actions results. If you have Jenkins configured, run the pipeline there.

If neither is configured, the local checks still practise the Terraform validation stage, but they do not prove that a remote CI pipeline works.

### Step 5: Clean up

If you want to remove the resources managed by this practice project:

```
terraform plan -destroy
```

Review the plan first. If it contains only the expected local practice resources:

```
terraform destroy
```

Type `yes` only after reviewing. Your `.tf` files and Git history remain available for revision.

## Module 5 — Interview revision

| Question                                           | Speakable answer                                                                           |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Why use Git with Terraform?                        | To track configuration changes, collaborate and review infrastructure updates.             |
| Why ignore state files?                            | They can contain sensitive infrastructure details and values.                              |
| What is CI?                                        | Automated checks that run when code changes.                                               |
| What is CD?                                        | Automated or controlled delivery and deployment of changes.                                |
| Why review a Terraform plan?                       | To understand what will be created, changed, replaced or destroyed.                        |
| Why use OIDC?                                      | To obtain temporary credentials without storing long-lived access keys.                    |
| What is least privilege?                           | Giving an identity only the permissions it needs.                                          |
| What happens when an apply fails?                  | Some changes may already have succeeded; inspect state and infrastructure before retrying. |
| Does Terraform automatically roll back everything? | No. Recovery depends on the failure and the infrastructure involved.                       |
| Why use a `Jenkinsfile`?                           | To define the CI/CD pipeline as code.                                                      |

## Terraform course completion checklist

You have now covered the five core modules:

- Module 1 — Fundamentals and workflow
- Module 2 — HCL, variables, outputs and resource logic
- Module 3 — AWS infrastructure
- Module 4 — State, modules, workspaces and imports
- Module 5 — Git, CI/CD and security

The checkmarks mean the material has been covered, not that every AWS deployment or CI workflow has been verified on your machine.

My recommendation for your junior DevOps preparation: prioritize being able to explain and demonstrate `init → fmt → validate → plan → apply`, resource dependencies, state, reusable modules, Git-based review, and safe AWS authentication. These are more useful early-career skills than memorizing advanced Terraform features.
