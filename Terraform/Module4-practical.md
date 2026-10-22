# Terraform Module 4 — Complete Hands-on Lab

We’ll practise Terraform state, reusable modules, resource renaming, drift, and remote-state concepts using your existing Ubuntu/WSL environment.

Goal: Finish with practical skills you can explain in a junior DevOps interview.

We’ll use a local Terraform lab first, so you can practise safely without creating additional AWS resources or incurring AWS charges. Remote S3 state will be covered separately because it needs an AWS S3 bucket and appropriate permissions.

## Lab roadmap

Lab 1 — Terraform state

Inspect resources, state and outputs.

Lab 2 — Reusable modules

Build and call a child module.

Lab 3 — Resource renaming

Practise a moved block without recreating the file.

Lab 4 — Drift and state operations

Observe changes and understand safe state management.

Lab 5 — Remote state

Learn S3 backend, locking, and safe migration.

## Lab 1 — Inspect your existing Terraform state

### Step 1: Open your project

Run:

```
cd ~/terraform-lab
pwd
ls -la
```

This shows your current directory and files.

### Step 2: Check whether Terraform has state

```
terraform state list
```

Possible outcomes:

- Resource addresses appear: Terraform has state entries for those resources.
- No state file or no state: You may not have applied resources in this project, or the backend/state may differ.
- An error appears: We’ll diagnose it before proceeding.

### Step 3: Inspect the current configuration

```
terraform validate
terraform output
```

`terraform output` may report that no outputs are defined. That is fine; it doesn't necessarily indicate a problem.

Don't run `terraform destroy`, delete `terraform.tfstate`, or reinitialize a different backend. First, let's see what your current project contains.

## Lab 2 — Build a reusable Terraform module

For the remaining labs, use a separate practice directory so we don't accidentally modify your AWS infrastructure.

### Step 1: Create the project

```
mkdir -p ~/terraform-state-module-lab/modules/note
mkdir -p ~/terraform-state-module-lab/output
cd ~/terraform-state-module-lab
```

### Step 2: Create the root configuration

```
nano main.tf
```

Paste:

```
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

module "note" {
  source      = "./modules/note"
  filename    = "${path.module}/output/note.txt"
  environment = "dev"
}

output "note_path" {
  value = module.note.file_path
}

resource "local_file" "old_name" {
  filename = "${path.module}/output/rename.txt"
  content  = "This file will demonstrate a moved block."
}
```

Save using `Ctrl+O`, press `Enter`, then `Ctrl+X`.

### Step 3: Create the child module

Create its variables file:

```
nano modules/note/variables.tf
```

```
variable "filename" {
  type = string
}

variable "environment" {
  type = string
}
```

Create the module's resource:

```
nano modules/note/main.tf
```

```
locals {
  content = "Environment: ${var.environment}\nCreated by Terraform module.\n"
}

resource "local_file" "note" {
  filename = var.filename
  content  = local.content
}
```

Create the module output:

```
nano modules/note/outputs.tf
```

```
output "file_path" {
  value = local_file.note.filename
}
```

### Step 4: Initialize, plan and apply

```
terraform init
terraform fmt -recursive
terraform validate
terraform plan
```

Review the plan. It should create two local files: the module-generated note and the root-level rename demonstration file.

Now apply:

```
terraform apply
```

Type `yes` after reviewing the plan.

Verify:

```
terraform state list
terraform output
cat output/note.txt
cat output/rename.txt
```

Expected learning: Your root configuration calls a child module, passes input variables, and receives an output. Terraform state tracks both the module resource and the root resource.

## Lab 3 — Practise resource renaming with `moved`

Imagine you renamed a resource in your code but wanted Terraform to keep managing the same underlying file.

### Step 1: Edit `main.tf`

```
nano main.tf
```

Change the resource name from `old_name` to `new_name`:

```
resource "local_file" "new_name" {
  filename = "${path.module}/output/rename.txt"
  content  = "This file will demonstrate a moved block."
}
```

Keep the module and output blocks unchanged.

### Step 2: Add a moved block

Add this block anywhere at the root level of `main.tf`:

```
moved {
  from = local_file.old_name
  to   = local_file.new_name
}
```

### Step 3: Review the plan

```
terraform fmt
terraform validate
terraform plan
```

Terraform should recognise the old address and move its state association to `local_file.new_name` instead of unnecessarily recreating the file.

If the plan looks correct:

```
terraform apply
terraform state list
```

Interview takeaway: A `moved` block communicates a resource-address change to Terraform. It is not a command to delete and recreate the resource.

## Lab 4 — Practise drift detection and state inspection

### Step 1: Inspect state

```
terraform state list
```

You should see addresses similar to:

```
local_file.new_name
module.note.local_file.note
```

Inspect one resource:

```
terraform state show module.note.local_file.note
```

This displays the resource attributes Terraform records.

### Step 2: Create drift

Change the generated file manually:

```
echo "Someone changed this file manually." > output/note.txt
```

Now run:

```
terraform plan
```

Terraform should detect that the file differs from its configured content and propose restoring the expected content, potentially by replacing the local-file resource.

### Step 3: Restore the expected configuration

```
terraform apply
cat output/note.txt
```

The file should again contain the content defined by your module.

Important: This demonstrates drift for a local file. AWS resources have different refresh and replacement behavior depending on the resource and changed attribute.

### Step 4: Practise changing a module input

Open `main.tf` and change:

```
environment = "dev"
```

to:

```
environment = "test"
```

Then run:

```
terraform plan
terraform apply
cat output/note.txt
```

You should see `Environment: test`.

This teaches you how module inputs affect the desired result and how Terraform detects the difference.

## Lab 5 — Remote state with S3

Remote state requires AWS access and a real S3 bucket. You can understand the workflow now without migrating your current AWS project.

### What you would configure

```
terraform {
  backend "s3" {
    bucket       = "your-globally-unique-state-bucket"
    key          = "terraform-state-module-lab/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

This belongs in the root configuration of the project whose state you want to store remotely.

Before initializing this backend, you must create or identify a suitable state bucket, configure permissions, and decide whether existing local state needs migration.

Then:

```
terraform init
```

If Terraform asks to migrate existing state, review the prompt carefully and preserve the existing state. Don't migrate your actual AWS project just for practice.

After migration, you can inspect the resources as usual:

```
terraform state list
terraform plan
```

### What to know for interviews

- Remote state enables controlled collaboration.
- S3 versioning can help recover earlier state versions.
- Native S3 locking with `use_lockfile = true` helps prevent conflicting concurrent operations.
- IAM permissions must restrict who can read or modify state.
- State can contain sensitive values, so protect it like confidential data.

## Lab 6 — Clean up the practice project

After verifying the labs, remove only the resources managed by this isolated local project:

```
cd ~/terraform-state-module-lab
terraform plan -destroy
```

Review the plan. If it only concerns your practice files and you no longer need them:

```
terraform destroy
```

Type `yes` after reviewing.

Confirm:

```
terraform state list
```

The state should no longer contain the lab's managed resources. You can remove the practice directory later if you want, but don't remove your original `~/terraform-lab` project.

## Final practical checklist

Hands-on progress

0/9

Created a child module and passed input variables

Used module outputs in the root configuration

Ran init, fmt, validate, plan and apply

Inspected state with list and show

Renamed a resource using a moved block

Changed a file manually and observed drift

Changed a module input and applied the update

Understood S3 remote state, locking and migration

Destroyed the isolated lab resources

You can mark a task complete after you actually perform and verify it. The checklist itself does not verify your terminal results.

Your next action: Start with Lab 2 and run `terraform init`, `terraform fmt -recursive`, `terraform validate`, and `terraform plan`. If any command fails, share the exact error and we’ll fix it before moving forward.
