You want all the actual Terraform code we've practised so far, gathered into one place — `main.tf`, `variables.tf`, `outputs.tf`, `count`, `for_each`, conditions, and related examples.

I'll organize them by topic so you can use them as revision notes. Each example is separate; don't paste every example into the same file, because some define resources with the same names.

# Terraform code collection — Lessons 1–2

## 1. Basic resource — `main.tf`

Creates a text file using Terraform's `local_file` resource.

```

resource "local_file" "hello" {
  filename = "${path.module}/devops.txt"
  content  = "I am learning DevOps"
}

```

Commands to run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

To view the file:

```
cat devops.txt
```

## 2. Variables — `variables.tf` and `main.tf`

`variables.tf`

```

variable "message" {
  type    = string
  default = "I am learning DevOps"
}

```

`main.tf`

```

resource "local_file" "hello" {
  filename = "${path.module}/devops.txt"
  content  = var.message
}

```

Here, `var.message` uses the variable value defined in `variables.tf`.

Override the default for a plan:

```
terraform plan -var='message=I am becoming a DevOps engineer'
```

## 3. Outputs — `outputs.tf`

With the resource named `local_file.hello`, you can output its path:

```

output "file_path" {
  value = local_file.hello.filename
}

```

To output its configured content instead:

```

output "file_content" {
  value = local_file.hello.content
}

```

- `.filename` → file path.
- `.content` → configured text content.

## 4. Locals and conditions — `main.tf`

```

variable "environment" {
  type    = string
  default = "dev"
}

locals {
  message = var.environment == "prod" ? "Production server" : "Development server"
}

output "message" {
  value = local.message
}

```

The `? :` expression means: if the condition is true, use the first value; otherwise, use the second.

- `prod` → `Production server`
- `dev` → `Development server`

## 5. `count` — create multiple files

`main.tf`

```

terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

variable "file_count" {
  type    = number
  default = 3
}

resource "local_file" "demo" {
  count    = var.file_count
  filename = "${path.module}/demo-${count.index}.txt"
  content  = "Hello from Terraform"
}

```

Expected files: `demo-0.txt`, `demo-1.txt`, `demo-2.txt`.

If `file_count = 5`, Terraform manages five instances of this resource.

## 6. `for_each` — create files from a map

`main.tf`

```

terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

variable "files" {
  type = map(string)

  default = {
    dev  = "Development environment"
    test = "Testing environment"
    prod = "Production environment"
  }
}

resource "local_file" "environment" {
  for_each = var.files

  filename = "${path.module}/${each.key}.txt"
  content  = each.value
}

```

Expected files:

| Filename   | Content                 |
| ---------- | ----------------------- |
| `dev.txt`  | Development environment |
| `test.txt` | Testing environment     |
| `prod.txt` | Production environment  |

- `each.key` → map key, such as `dev`.
- `each.value` → corresponding value, such as `Development environment`.

## 7. Lists with `length()` and `count.index`

`main.tf`

```

terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

variable "environments" {
  type    = list(string)
  default = ["dev", "test", "prod"]
}

resource "local_file" "environment" {
  count = length(var.environments)

  filename = "${path.module}/${var.environments[count.index]}.txt"
  content  = "Environment: ${var.environments[count.index]}"
}

```

Here, `length()` counts the list items, while `count.index` selects the current item's position.

## 8. Combined practice — Docker, Jenkins and Terraform

This is the exercise we were working toward. Try writing it yourself first.

`main.tf`

```

terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

variable "tools" {
  # Define a list of strings
}

resource "local_file" "tool" {
  # Use count and count.index
  # Create one .txt file per tool
}

```

Requirements:

- List: `["Docker", "Jenkins", "Terraform"]`
- Use `count = length(var.tools)`.
- Filename: tool name followed by `.txt`.
- Content: the matching tool name.

## 9. Quick command reference

| Command              | Purpose                                                 |
| -------------------- | ------------------------------------------------------- |
| `terraform init`     | Initialize the working directory and download providers |
| `terraform fmt`      | Format Terraform files                                  |
| `terraform validate` | Check configuration syntax and consistency              |
| `terraform plan`     | Preview proposed changes                                |
| `terraform apply`    | Apply proposed changes                                  |
| `terraform output`   | Display declared outputs                                |
| `terraform destroy`  | Destroy managed resources                               |

Important: `terraform destroy` can delete resources, so review the plan carefully before confirming. For your local-file exercises, it may remove files managed by Terraform.

That's the code we've covered so far, gathered into one revision collection. The next concepts in our shortened plan are `for_each` practice, then functions and dependencies.
