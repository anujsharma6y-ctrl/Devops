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
Let's continue with Lesson 3: Terraform Functions and Resource Dependencies. We'll keep it short and practical for junior DevOps.

# Lesson 3 — Functions and dependencies

## 1. Terraform functions

Functions help you process values without writing extra resource blocks.

| Function    | What it does                  | Example result                             |
| ----------- | ----------------------------- | ------------------------------------------ |
| `length()`  | Counts items or characters    | `length(["Docker","Git"])` → `2`           |
| `upper()`   | Converts text to uppercase    | `upper("devops")` → `"DEVOPS"`             |
| `lower()`   | Converts text to lowercase    | `lower("AWS")` → `"aws"`                   |
| `join()`    | Combines list items into text | `join(",", ["dev","prod"])` → `"dev,prod"` |
| `element()` | Gets an item from a list      | `element(["a","b"], 1)` → `"b"`            |

## 2. Practice code — `main.tf`

This example uses `upper()`, `length()`, and `join()`.

```

variable "tools" {
  type    = list(string)
  default = ["docker", "jenkins", "terraform"]
}

locals {
  tool_count   = length(var.tools)
  capital_tools = [for tool in var.tools : upper(tool)]
  tool_message = join(", ", local.capital_tools)
}

output "total_tools" {
  value = local.tool_count
}

output "tool_names" {
  value = local.tool_message
}

```

Run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
```

Expected outputs:

```
total_tools = 3
tool_names = "DOCKER, JENKINS, TERRAFORM"
```

Note: This example defines no infrastructure resources, so `terraform apply` only evaluates the configuration and records outputs.

## 3. Resource references and dependencies

Imagine Terraform must create a file first, then create another file containing the first file's path.

```

resource "local_file" "first" {
  filename = "${path.module}/first.txt"
  content  = "First file"
}

resource "local_file" "second" {
  filename = "${path.module}/second.txt"
  content  = "The first file is at: ${local_file.first.filename}"
}

```

The important expression is:

```
local_file.first.filename
```

It means: access the `filename` attribute of the resource named `first`.

Terraform detects this reference and automatically creates an implicit dependency. It knows that the first resource must be processed before the second.

### When do you use `depends_on`?

Use it when a dependency exists but Terraform cannot infer it from a resource reference.

```
resource "local_file" "second" {
  filename = "${path.module}/second.txt"
  content  = "Second file"

  depends_on = [local_file.first]
}
```

Normally, prefer implicit dependencies through references. Add `depends_on` only when necessary.

## 4. One interview question

You have two Terraform resources. Resource B uses `local_file.a.filename` in its configuration. What does Terraform do?

Creates B before A

Automatically detects the dependency and processes A first

Always creates both at exactly the same time

Requires `depends_on` in every case

Check answer

After this, we'll move to Lesson 4: Terraform modules—a key skill for reusable infrastructure code.
