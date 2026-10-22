Let's slow down and learn Terraform Module 2 again from the beginning, using simple English + Hinglish, small examples, and one concept at a time. You don't need to memorize all the syntax right now.

The main problem with the previous explanation is that it introduced too many Terraform concepts together. We'll understand what each one does first, then connect them.

# Terraform Module 2 — Part 1: Understand the basics

## 1. What are we trying to do?

Imagine you want Terraform to create a text file on your computer.

Without Terraform, you might run:

```
touch app.txt
```

But this only creates an empty file. Now imagine you want Terraform to create the file and put some text inside it.

You write instructions in a file called `main.tf`:

```
resource "local_file" "app" {
  filename = "app.txt"
  content  = "Hello DevOps"
}
```

Then you run:

```
terraform init
terraform plan
terraform apply
```

Terraform creates `app.txt` containing `Hello DevOps`.

main.tf

Your instructions

Terraform

Reads instructions and manages changes

app.txt

Hello DevOps

This small example is the foundation for understanding Terraform configuration.

## 2. What does `resource` mean?

Look at this line:

```
resource "local_file" "app" {
```

Break it into three parts:

| Part           | Simple meaning                                |
| -------------- | --------------------------------------------- |
| `resource`     | I want Terraform to manage something          |
| `"local_file"` | What kind of thing? A file                    |
| `"app"`        | What name will I use for it inside Terraform? |

Think of it like ordering something:

- Resource type: what you want.
- Resource name: your label for it.
- Arguments: the details of what you want.

For example, in AWS you might eventually write:

```
resource "aws_s3_bucket" "photos" {
  bucket = "my-unique-bucket-name"
}
```

Here, `aws_s3_bucket` means an S3 bucket, and `photos` is its Terraform name. This is an illustrative example; the bucket name must be globally unique.

Important: `photos` is not the actual bucket name. The value assigned to `bucket` is the actual bucket name.

## 3. What are `filename` and `content`?

Look at the full block:

```
resource "local_file" "app" {
  filename = "app.txt"
  content  = "Hello DevOps"
}
```

- `filename` tells Terraform where the file should be.
- `content` tells Terraform what text the file should contain.
- `=` assigns a value.
- `{ }` contains the resource's instructions.

Hinglish: `filename` matlab file ka naam/path, aur `content` matlab file ke andar kya likhna hai.

Notice that `filename` and `content` are not special Terraform commands. They are arguments accepted by this particular resource type.

## 4. What is a variable?

Imagine you have three environments:

- Development: `dev`
- Testing: `test`
- Production: `prod`

You want your Terraform configuration to work for all three environments.

Without a variable, you might repeatedly edit `"dev"` to `"test"` or `"prod"` in your code.

A variable lets you provide a value that can be changed without rewriting the whole configuration.

Example:

```
variable "environment" {
  type    = string
  default = "dev"
}
```

Let's understand every line:

| Code              | Meaning                                 |
| ----------------- | --------------------------------------- |
| `variable`        | Declare an input                        |
| `"environment"`   | Name of the input                       |
| `type = string`   | The value should be text                |
| `default = "dev"` | Use `dev` if no other value is provided |

Now use that variable:

```
resource "local_file" "app" {
  filename = "app.txt"
  content  = "Environment: ${var.environment}"
}
```

The important part is:

```
${var.environment}
```

It means: insert the value of the `environment` variable here.

If the value is `dev`, the file contains:

```
Environment: dev
```

If the value changes to `prod`, it contains:

```
Environment: prod
```

Hinglish: Variable ek changeable value hai. Ek baar code likho, phir input badal kar alag environment ke liye use karo.

### Try to understand this example

Choose an environment

dev

test

prod

Terraform file content would be:

`Environment: {envChoice}`

Changing the variable changes the resulting content.

## 5. What are locals?

Suppose you have an environment variable with the value `dev`.

You want to build a longer name from it, such as `web-dev`.

You can use a local value:

```
locals {
  app_name = "web-${var.environment}"
}
```

Now:

- `var.environment` is the input.
- `local.app_name` is the calculated value.

If the environment is `dev`, then `local.app_name` becomes `web-dev`.

Think of locals as a way to give a name to a value or calculation you want to reuse.

Example:

```
locals {
  app_name = "web-${var.environment}"
}

resource "local_file" "app" {
  filename = "${local.app_name}.txt"
  content  = "My application"
}
```

For `dev`, Terraform manages a file named `web-dev.txt`.

Remember the difference:

| Syntax            | Meaning                |
| ----------------- | ---------------------- |
| `var.environment` | Read an input variable |
| `local.app_name`  | Read a local value     |

You do not need locals for every small calculation. Use them when a value is useful to name or reuse.

## 6. What are outputs?

After Terraform creates something, you may want it to show you a useful result.

For example:

```
output "app_filename" {
  value = local_file.app.filename
}
```

This tells Terraform to display the filename attribute of the resource named `app`.

After applying the configuration, you can run:

```
terraform output
```

Terraform can display the output value.

The three concepts now fit together:

Variable

Input: `dev`

Local

Calculates: `web-dev`

Resource

Creates `web-dev.txt`

Output

Shows a useful result

- Variable: value comes in.
- Local: value is calculated or reused.
- Resource: Terraform manages the object.
- Output: Terraform displays a result.

You will see these four concepts repeatedly in real Terraform projects.

# Part 2: How do we provide values and create multiple resources?

## 7. What is a `.tfvars` file?

Earlier, we wrote this:

```
variable "environment" {
  type    = string
  default = "dev"
}
```

The value `dev` is the default. But suppose you want to choose `prod` without changing `main.tf`.

Create another file named `terraform.tfvars`:

```
environment = "prod"
```

Now Terraform uses `prod` instead of the default `dev`.

Think of the files this way:

| File               | Purpose                                 |
| ------------------ | --------------------------------------- |
| `main.tf`          | Defines what Terraform should manage    |
| `variables.tf`     | Common place to declare input variables |
| `terraform.tfvars` | Supplies input values                   |
| `outputs.tf`       | Common place to declare outputs         |

These are naming conventions, not mandatory separate files. Terraform reads `.tf` files in the same directory together.

Remember: A `.tfvars` file supplies values; it does not create infrastructure by itself.

## 8. What are variable types?

A type tells Terraform what kind of value a variable accepts.

| Type           | Example           | Used for             |
| -------------- | ----------------- | -------------------- |
| `string`       | `"dev"`           | Text                 |
| `number`       | `3`               | Numbers              |
| `bool`         | `true`            | True/false           |
| `list(string)` | `["dev", "prod"]` | Ordered list of text |
| `map(string)`  | `{env = "dev"}`   | Key-value pairs      |

For example:

```
variable "instance_count" {
  type    = number
  default = 2
}
```

This variable expects a number, such as `2`, not the text `"two"`.

You don't need to memorize every possible Terraform type yet. Understand the common ones and recognize them when reading code.

## 9. What are expressions and functions?

An expression is code that produces a value.

For example:

```
"web-${var.environment}"
```

If the environment is `dev`, the result is `web-dev`.

A function takes an input and returns a result, like a calculator.

```
lower("PROD")
```

Result: `prod`.

Other useful functions:

| Function                    | What it does      |
| --------------------------- | ----------------- |
| `upper("dev")`              | Returns `DEV`     |
| `length(["a", "b"])`        | Returns `2`       |
| `join("-", ["web", "dev"])` | Returns `web-dev` |

You don't need to memorize a huge list of functions. Learn them as you encounter real use cases.

### What is a conditional expression?

A conditional selects one of two values:

```
var.environment == "prod" ? "large" : "small"
```

Read it as:

> If the environment equals `prod`, use `large`; otherwise, use `small`.

The `?` separates the condition from the true result, and `:` introduces the false result.

This is useful when different environments need different settings.

## 10. What are `count` and `for_each`?

Suppose you want Terraform to create three files.

You could write three separate resource blocks, but that repeats code. Terraform offers two ways to create multiple instances from one resource block.

### Option A: `count`

```
resource "local_file" "report" {
  count    = 3
  filename = "report-${count.index}.txt"
  content  = "Hello"
}
```

This creates three files:

- `report-0.txt`
- `report-1.txt`
- `report-2.txt`

`count = 3` means three instances. `count.index` provides the index, starting at zero.

### Option B: `for_each`

```
resource "local_file" "report" {
  for_each = toset(["dev", "prod"])
  filename = "${each.key}.txt"
  content  = "Environment: ${each.key}"
}
```

This creates:

- `dev.txt`
- `prod.txt`

`each.key` represents the current item.

### Which should you use?

Use `count`

When you need a certain number of similar instances.

Example: create 3 identical test files.

Use `for_each`

When each instance has a meaningful name or key.

Example: create separate resources for `dev`, `test` and `prod`.

Why does this matter? With `count`, removing an item from the middle of an indexed list can shift indexes and cause unwanted changes. With `for_each`, named keys help Terraform track each instance by identity.

# Part 3: Reading existing resources and managing changes

## 11. Resource vs data source

This is particularly important for AWS.

Imagine you need an Amazon Machine Image (AMI) ID to create an EC2 instance.

You have two different tasks:

- Task A: Create an EC2 instance.
- Task B: Find an existing AMI that the instance can use.

Task A uses a resource. Task B can use a data source.

|         | Resource                   | Data source                               |
| ------- | -------------------------- | ----------------------------------------- |
| Purpose | Create or manage an object | Read information about an existing object |
| Keyword | `resource`                 | `data`                                    |
| Example | Create an EC2 instance     | Look up an AMI                            |

Simplified example:

```
data "aws_ami" "selected" {
  # Search criteria go here
}
```

The `data` block reads information; it does not create the AMI.

You can use the returned ID in an EC2 resource later. The actual lookup requires valid search criteria and an AWS provider configuration.

## 12. What is a dependency?

Imagine Terraform needs to create a network before creating a server that uses that network.

The order should be:

```
Create network
     ↓
Create server using that network
```

If the server configuration references the network's ID, Terraform normally detects this dependency automatically.

That is called an implicit dependency.

Sometimes a dependency exists but is not obvious from a resource reference. Then you may need `depends_on`, which explicitly tells Terraform which object must be handled first.

Junior DevOps rule: Let Terraform infer dependencies from references whenever possible. Use `depends_on` only when necessary.

## 13. What are lifecycle rules?

Lifecycle rules tell Terraform how to handle certain changes to a resource.

Three rules are especially useful to recognize:

| Rule                    | Simple meaning                                                               |
| ----------------------- | ---------------------------------------------------------------------------- |
| `create_before_destroy` | Try to create a replacement before removing the old resource                 |
| `prevent_destroy`       | Reject a Terraform plan that destroys the resource while the rule is present |
| `ignore_changes`        | Ignore selected attribute changes when planning updates                      |

For example:

```
lifecycle {
  prevent_destroy = true
}
```

This can help prevent accidental Terraform-planned deletion of an important resource.

But it is not a backup and is not a universal guarantee against deletion. If you remove the resource block from your configuration, the protection may no longer apply to the resulting plan.

You do not need to use all three rules in every project. First understand what each one is meant to do.

## 14. What does resource replacement mean?

Suppose Terraform manages a resource and you change a setting.

Sometimes Terraform can update that setting in place. Other times, the platform requires Terraform to destroy the old object and create a replacement.

Terraform's plan can show `-/+` for a replacement.

The practical lesson is simple:

1. Run `terraform plan`.
2. Read the proposed changes.
3. Check whether anything will be replaced or destroyed.
4. Apply only when you understand the impact.

Replacement can cause downtime or data loss depending on the resource, so never approve a plan blindly.

# Final recap — remember these 8 things

| Concept    | Remember it as                            |
| ---------- | ----------------------------------------- |
| `resource` | Create or manage something                |
| `variable` | Input value                               |
| `local`    | Reusable or calculated value              |
| `output`   | Display a result                          |
| `.tfvars`  | Supply variable values                    |
| `count`    | Create a number of instances              |
| `for_each` | Create instances from named items         |
| `data`     | Read information about an existing object |

And three more:

- `depends_on` = explicitly specify a dependency.
- `lifecycle` = influence how Terraform handles changes.
- `terraform plan` = preview proposed changes before applying them.

## Let's check your understanding first

Don't worry about getting the answers wrong. I want to identify exactly which concept needs more explanation before we move to the practical lab.

1\. You want to give Terraform the environment name `prod`. Which concept is used for this input?

Variable

Local

Output

2\. You want Terraform to create five similar files using one resource block. Which would you most likely use?

Data source

Count

Output

3\. You want Terraform to find an existing AMI instead of creating one. Which do you use?

Resource

Variable

Data source

Check my understanding

We'll make sure these basics are clear before moving on. You don't need to rush through Terraform.
You got 2 out of 3 correct! Good progress. You understand variables and `count`. We only need to fix your understanding of resources vs data sources.

## Your results

1\. Variable — Correct

You use a variable to supply an input such as `prod`.

```
variable "environment" {
  default = "prod"
}
```

2\. `count` — Correct

`count = 5` tells Terraform to create five instances of a resource block.

```
resource "local_file" "report" {
  count    = 5
  filename = "report-${count.index}.txt"
  content  = "Hello"
}
```

3\. Resource — Incorrect

The correct answer is data source when you want to find information about an existing AMI.

Let's make this distinction clear.

## The concept to fix: Resource vs data source

Imagine AWS already has an AMI (a machine image you can use to launch an EC2 instance).

There are two different tasks:

Resource

Create or manage something

Example: You want Terraform to create an EC2 instance.

```
resource "aws_instance" "web" {
  # EC2 configuration
}
```

Hinglish: Nayi cheez banana ya existing resource ko manage karna.

Data source

Find or read information about something that already exists

Example: You want Terraform to look up an AMI ID.

```
data "aws_ami" "selected" {
  # AMI search criteria
}
```

Hinglish: Jo cheez already available hai, uski information nikalna.

### How they work together in a real AWS task

1\. Data source

Find an existing AMI and obtain its ID

2\. Resource

Use that AMI ID to create an EC2 instance

Interview shortcut: `data` = read; `resource` = manage.

One important detail: a resource does not always mean creating something brand new. Terraform can also manage an existing object that has been imported into its state.

## One quick check

Scenario: An S3 bucket already exists in your AWS account. You only want Terraform to read its region or other available information, not create the bucket. Which would you use?

A. `resource`

B. `data`

C. `variable`

Check answer

We will focus only on this distinction for now. Your other two answers were correct, so you don't need to relearn those concepts.
