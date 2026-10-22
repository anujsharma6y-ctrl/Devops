✅ Correct! Answer B.

Terraform automatically detects the dependency because Resource B references `local_file.a.filename`.

```
resource "local_file" "a" {
  filename = "${path.module}/a.txt"
  content  = "File A"
}

resource "local_file" "b" {
  filename = "${path.module}/b.txt"
  content  = local_file.a.filename
}
```

Remember:

- Implicit dependency: Terraform detects it through a resource reference.
- Explicit dependency: Use `depends_on` when Terraform cannot infer the dependency.
- Terraform creates A before B in this example.

# Lesson 4 — Terraform Modules

A module is a reusable collection of Terraform configuration files.

For example, instead of writing the same EC2 configuration repeatedly, you can create a module and reuse it for development, testing, and production.

## 1. Understand the folder structure

terraform-project/

`main.tf`

Root configuration

`variables.tf`

modules/

hello-file/

`main.tf` `variables.tf` `outputs.tf`

The root module calls the child module. The child module contains the reusable code.

## 2. Create a simple module

Open your Ubuntu terminal:

```
mkdir -p ~/terraform-lesson4/modules/hello-file
cd ~/terraform-lesson4
```

Create the child module file:

```
nano modules/hello-file/main.tf
```

Add:

```

variable "filename" {
  type = string
}

variable "message" {
  type = string
}

resource "local_file" "this" {
  filename = "${path.module}/${var.filename}"
  content  = var.message
}

output "file_path" {
  value = local_file.this.filename
}

```

Save the file.

## 3. Call the module from the root

Create or edit the root file:

```
nano main.tf
```

Add:

```

module "hello" {
  source   = "./modules/hello-file"
  filename = "hello.txt"
  message  = "Learning Terraform modules"
}

output "created_file_path" {
  value = module.hello.file_path
}

```

Here:

- `source` tells Terraform where the module lives.
- `filename` and `message` are inputs passed into the module.
- `module.hello.file_path` reads the module's output.

Run:

```
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
```

Important: The file is created inside the child module's directory because `path.module` refers to that module's directory. So check it with:

```
cat modules/hello-file/hello.txt
```

## Your turn

Why do we use a Terraform module?

To replace Terraform state completely

To reuse and organize Terraform configuration

To automatically remove all AWS resources

Check answer
