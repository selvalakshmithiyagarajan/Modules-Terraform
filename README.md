# Terraform_Modules
Production Level Terraform  Deployment Modules 
![TM](https://github.com/saikiranpi/Terraform_Modules/assets/109568252/088be80e-2840-44e8-8b33-e4ba835e501f)
In the dynamic landscape of AWS resource management, catering to the diverse needs of QA, Dev, and production environments can be quite a juggling act. Rather than embarking on a daunting quest to rewrite or edit all Terraform code in one monolithic sweep, a smarter approach emerges: the strategic use of Terraform modules. These modular building blocks allow teams to sculpt their infrastructure desires independently. QA teams can summon specific resources, the Dev team can beckon different services, and the production environment can luxuriate in its unique set of resources. It's a symphony of flexibility, enabling each environment to dance to its own AWS tune while maintaining order and efficiency.
Terraform modules diagram is divided into two main sections: production and development. Each section contains a set of modules that are specific to that environment.
The production section contains modules for our core infrastructure, such as our network, load balancers, IAM roles, and NAT gateways. The development section contains modules for our development and testing infrastructure, such as our development VPCs and subnets

🚀 Terraform Workspaces for Dev & Prod Environment Infrastructure
This repository contains a production-ready Terraform setup that efficiently manages multiple isolated AWS environments (Dev and Prod) using Terraform Workspaces. Each environment maintains independent infrastructure with its own state file, stored in an S3 bucket, and protected with DynamoDB-based state locking.

🧰 Key Features
Isolated Environments: Separate workspaces for Dev and Prod environments using a single Terraform codebase.

Centralized State Management: Terraform state stored in S3 with workspace-based key separation.

State Locking: Prevents concurrent Terraform operations using DynamoDB.

Environment-Specific Variables: Each environment has its own .tfvars file to manage config like server names, tags, or AMIs.

Scalable & Reusable Modules: Uses modular code for EC2, VPC, IAM, etc.

📋 Prerequisites
Terraform v1.4 or later

AWS CLI configured with IAM permissions

One S3 bucket created (e.g., terraform-prod-state)

DynamoDB table created (e.g., terraform-locks with LockID as the primary key)

🗂️ Project Structure
bash
Copy
Edit
.
├── backend.tf
├── main.tf
├── variables.tf
├── dev.tfvars
├── prod.tfvars
├── ec2.tf
├── outputs.tf
└── modules/
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
🔁 Environment Setup via Workspaces
We use Terraform workspaces to isolate infrastructure deployments.

1. Create & Switch Workspaces
bash
Copy
Edit
terraform workspace new dev
terraform workspace new prod
terraform workspace list
2. Initialize Backend with S3 & DynamoDB
hcl
Copy
Edit
# backend.tf
terraform {
  backend "s3" {
    bucket         = "terraform-prod-state"
    key            = "env/${terraform.workspace}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}
Note: ${terraform.workspace} dynamically places state files in different folders (dev, prod).

🧪 Deploying Environments
For Development Environment:
bash
Copy
Edit
terraform workspace select dev
terraform init
terraform plan -var-file=dev.tfvars
terraform apply -var-file=dev.tfvars
For Production Environment:
bash
Copy
Edit
terraform workspace select prod
terraform init
terraform plan -var-file=prod.tfvars
terraform apply -var-file=prod.tfvars
🛠️ Custom EC2 Configuration per Environment
Inside ec2.tf or module:

hcl
Copy
Edit
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  tags = {
    Name = "${var.env}-web-instance"
  }
  user_data = <<-EOF
              #!/bin/bash
              echo "Welcome to ${var.env} environment!" > /var/www/html/index.html
              EOF
}
🔐 State Locking with DynamoDB
To prevent race conditions during deployments:

hcl
Copy
Edit
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
Apply once, or manage outside of Terraform and exclude using:

bash
Copy
Edit
terraform state rm aws_dynamodb_table.terraform_lock
📤 Push to GitHub
bash
Copy
Edit
git init
git add .
git commit -m "Initial commit for multi-env Terraform setup"
git remote add origin https://github.com/your-username/terraform-envs.git
git push -u origin main
💣 Destroying Environments
To clean up:

bash
Copy
Edit
terraform workspace select dev
terraform destroy -var-file=dev.tfvars

terraform workspace select prod
terraform destroy -var-file=prod.tfvars
To remove workspaces:

bash
Copy
Edit
terraform workspace delete dev
terraform workspace delete prod
✅ Summary
This setup helps you:

Manage Dev & Prod AWS environments from one codebase

Avoid cross-environment state conflicts

Easily switch contexts with Terraform Workspaces

Maintain environment isolation, traceability, and scalability
