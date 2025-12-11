# 🔐 IAM with Terraform Lab (Using LocalStack)

In this lab, I learned how to **deploy IAM resources using Terraform**, configure the AWS provider to work with **LocalStack**, fix provider authentication issues, and dynamically create multiple IAM users using the **count** meta-argument. This lab mirrors real AWS IAM provisioning but uses a mock environment for safe experimentation.

---

## 📋 Lab Overview

**Goal:**

* Use Terraform to provision IAM users
* Configure the AWS provider to work with LocalStack
* Understand how endpoint overrides work in a mock environment
* Use `count` + list variables to dynamically create multiple IAM users

**Learning Outcomes:**

* Write Terraform resource blocks for IAM users
* Fix provider configuration errors
* Use `provider.tf` to define AWS provider settings
* Use `count = length(var.list)` to create multiple resources
* Fetch variable values from `variables.tf`

---

## 🛠 Step-by-Step Journey

### Step 1: Create IAM User Resource (Mary)

**Directory:**

```
/root/terraform-projects/IAM
```

Created file:

```
iam-user.tf
```

Resource block:

```hcl
resource "aws_iam_user" "users" {
  name = "Mary"
}
```

---

### Step 2: Initialize Terraform

```bash
terraform init
terraform plan
```

**Plan failed** due to:

❌ Invalid AWS region
❌ Missing provider configuration

---

### Step 3: Create a provider.tf File

A new file was created:

```
provider.tf
```

Added AWS provider configuration:

```hcl
provider "aws" {
  region = "ca-central-1"
}
```

Running `terraform plan` again:

❌ Failed — invalid credentials for mock environment

---

### Step 4: Update Provider Block for LocalStack

The lab automatically updated the file to include mock configurations:

```hcl
provider "aws" {
  region                      = "ca-central-1"
  access_key                  = "mock"
  secret_key                  = "mock"
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  s3_force_path_style         = true
  endpoints {
    iam = "http://aws:4566"
  }
}
```

This is only needed for LocalStack labs — not real AWS.

---

### Step 5: Run Terraform Again

```bash
terraform plan
terraform apply
# yes
```

✔️ IAM user **Mary** was created successfully.

---

### Step 6: Create Additional IAM Users Using Variables + Count

The file `variables.tf` was added containing:

```hcl
variable "project-sapphire-users" {
  type    = list(string)
  default = ["Alice", "Bob", "Charlie", "Dan", "Eve"]
}
```

**Questions Answered:**

| Question       | Answer                   |
| -------------- | ------------------------ |
| Variable name? | `project-sapphire-users` |
| Data type?     | `list(string)`           |

---

### Step 7: Update iam-user.tf to Create All Users in the List

Updated resource block:

```hcl
resource "aws_iam_user" "users" {
  count = length(var.project-sapphire-users)

  name = var.project-sapphire-users[count.index]
}
```

This loops through the list and creates one IAM user per entry.

Commands executed:

```bash
terraform init
terraform plan
terraform apply
# yes
```

✔️ All Project Sapphire users created successfully.

---

## ✅ Key Commands Summary

| Task                         | Command                                 |
| ---------------------------- | --------------------------------------- |
| Initialize Terraform         | `terraform init`                        |
| Preview resource creation    | `terraform plan`                        |
| Apply changes                | `terraform apply`                       |
| Define AWS provider          | `provider "aws" { ... }`                |
| Add LocalStack endpoint      | `endpoints { iam = "http://aws:4566" }` |
| Loop through list with count | `count = length(var.list)`              |
| Access list element          | `var.list[count.index]`                 |

---

## 💡 Notes / Tips

* LocalStack requires **endpoint overrides**, otherwise the provider tries to authenticate against real AWS.
* When using mock credentials, always include:

  * `skip_credentials_validation = true`
  * `skip_metadata_api_check = true`
* `count` is perfect for creating multiple IAM users from a simple list.
* Always store provider configuration separately in `provider.tf`.
* `variables.tf` helps scale configurations cleanly.

---

## 📌 Lab Summary

| Step                               | Status | Key Takeaways                              |
| ---------------------------------- | ------ | ------------------------------------------ |
| Create IAM user Mary               | ✅      | First resource created via Terraform       |
| Fix provider errors                | ✅      | Added provider.tf + LocalStack settings    |
| Successfully applied configuration | ✅      | Mary created in mock IAM                   |
| Inspect variables.tf               | ✅      | Found list(string) variable                |
| Use count to create multiple users | ✅      | Dynamically created Project Sapphire users |

---

## ✅ References

* [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
* [Terraform `count` Meta-Argument](https://developer.hashicorp.com/terraform/language/meta-arguments/count)
* [LocalStack AWS Emulation](https://docs.localstack.cloud/)
* [IAM User Resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user)
