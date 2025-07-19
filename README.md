# Terraform_Modules
Production Level Terraform  Deployment Modules 
![TM](https://github.com/saikiranpi/Terraform_Modules/assets/109568252/088be80e-2840-44e8-8b33-e4ba835e501f)
In the dynamic landscape of AWS resource management, catering to the diverse needs of QA, Dev, and production environments can be quite a juggling act. Rather than embarking on a daunting quest to rewrite or edit all Terraform code in one monolithic sweep, a smarter approach emerges: the strategic use of Terraform modules. These modular building blocks allow teams to sculpt their infrastructure desires independently. QA teams can summon specific resources, the Dev team can beckon different services, and the production environment can luxuriate in its unique set of resources. It's a symphony of flexibility, enabling each environment to dance to its own AWS tune while maintaining order and efficiency.
Terraform modules diagram is divided into two main sections: production and development. Each section contains a set of modules that are specific to that environment.
The production section contains modules for our core infrastructure, such as our network, load balancers, IAM roles, and NAT gateways. The development section contains modules for our development and testing infrastructure, such as our development VPCs and subnets
# Terraform Workspaces for Multi-Environment Infrastructure

This repository demonstrates how to manage multiple identical environments (Dev, UAT, and Prod) using **Terraform Workspaces**. Each environment creates its own set of resources (like EC2 instances) with unique names, and uses a separate state file in an S3 backend with DynamoDB for locking.

> Tools Used: Visual Studio Code for editing, Git Bash for Terraform operations.

---

## ⚖️ Prerequisites

* Terraform installed (`>= 1.0`)
* AWS CLI configured with IAM permissions
* S3 bucket for backend
* DynamoDB table for state locking

---

## 🕛 Infrastructure Overview

You will create three environments:

* **Dev**: 3 EC2 instances
* **UAT**: 3 EC2 instances
* **Prod**: 3 EC2 instances

Each environment uses its own `.tfvars` file and is isolated using Terraform workspaces.

---

## ✅ Step-by-Step Guide

### 1. Clone the Repo

```bash
git clone https://github.com/your-username/terraform-multi-env.git
cd terraform-multi-env
```

### 2. Setup Remote Backend

Create an S3 bucket and DynamoDB table for storing state files and locks.

Sample `main.tf` backend config:

```hcl
terraform {
  backend "s3" {
    bucket         = "your-s3-bucket"
    key            = "env/${terraform.workspace}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
  }
}
```

### 3. Create `.tfvars` Files

* `dev.tfvars`
* `uat.tfvars`
* `prod.tfvars`

Each file can define values like:

```hcl
env = "dev"
instance_count = 3
```

### 4. Initialize and Format

```bash
terraform init
terraform validate
terraform fmt
```

### 5. Create and Select Workspace

```bash
terraform workspace new dev
terraform workspace new uat
terraform workspace new prod
terraform workspace select dev
```

### 6. Apply Configuration

```bash
terraform apply -var-file=dev.tfvars
terraform workspace select uat
terraform apply -var-file=uat.tfvars
terraform workspace select prod
terraform apply -var-file=prod.tfvars
```

### 7. Add EC2 Instances in `ec2.tf`

```hcl
resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "${var.env}-Server-${count.index + 1}"
  }

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello from ${var.env}" > /var/www/html/index.html
              EOF
}
```

### 8. Switch Between Environments

```bash
terraform workspace select dev
terraform plan -var-file=dev.tfvars
terraform apply -var-file=dev.tfvars
```

Repeat for UAT and Prod.

### 9. Get Output (e.g., Public IPs)

```bash
terraform output
```

### 10. Destroy Infrastructure

```bash
terraform workspace select prod
terraform destroy -var-file=prod.tfvars
```

### 11. Delete Workspaces

```bash
terraform workspace delete dev
terraform workspace delete uat
terraform workspace delete prod
```

---

## 📊 State Locking with DynamoDB

`dynamodb.tf`:

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

Run:

```bash
terraform apply -target=aws_dynamodb_table.terraform_locks
```

To exclude it from future state management:

```bash
terraform state rm aws_dynamodb_table.terraform_locks
```

---

## 📝 Push Code to GitHub

```bash
git init
git add .
git commit -m "Initial Terraform Multi-Environment Setup"
git remote add origin https://github.com/your-username/terraform-multi-env.git
git push -u origin main
```

---

## 🌟 Conclusion

This setup uses **Terraform Workspaces** to create and manage multiple isolated environments with minimal duplication. State is securely stored in **S3**, and locked via **DynamoDB**.

Feel free to extend it with modules, autoscaling, VPCs, or load balancers as your infrastructure grows!
