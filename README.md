# Terraform_anugula

A Terraform module is just a reusable, self-contained set of .tf files — a folder with resources, variables, and outputs — that you can call from other configurations. Every Terraform config you write is technically already a "root module"; when people say "create a module," they usually mean a reusable child module.

1. Basic folder structure
modules/
  ec2-instance/
    main.tf
    variables.tf
    outputs.tf
    versions.tf   (optional)
2. Define inputs — variables.tf
hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "ami" {
  description = "AMI ID"
  type        = string
}

variable "name" {
  description = "Name tag"
  type        = string
}
3. Define resources — main.tf
hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = var.name
  }
}
4. Define outputs — outputs.tf
hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
5. Use the module from your root config
hcl
module "web_server" {
  source        = "./modules/ec2-instance"
  ami           = "ami-0123456789"
  instance_type = "t3.small"
  name          = "web-1"
}

output "web_ip" {
  value = module.web_server.public_ip
}

source can point to:

A local path: "./modules/ec2-instance"
A Git repo: "git::https://github.com/org/repo.git//modules/ec2"
Terraform Registry: "terraform-aws-modules/vpc/aws"
An S3/GCS bucket, etc.
6. Reusing the module multiple times
hcl
module "web_server_1" {
  source        = "./modules/ec2-instance"
  ami           = "ami-0123456789"
  name          = "web-1"
}

module "web_server_2" {
  source        = "./modules/ec2-instance"
  ami           = "ami-0123456789"
  name          = "web-2"
}

Or combined with for_each on the module block itself (Terraform 0.13+):

hcl
variable "servers" {
  default = {
    web1 = "t3.micro"
    web2 = "t3.small"
  }
}

module "web_server" {
  source        = "./modules/ec2-instance"
  for_each      = var.servers
  ami           = "ami-0123456789"
  instance_type = each.value
  name          = each.key
}

Reference specific outputs: module.web_server["web1"].public_ip

7. Initialize

Whenever you add/change a module source, run:

bash
terraform init

This downloads/links the module before you can plan/apply.

Good practices for writing modules

Practice	Why

Keep modules focused on one purpose (e.g., "vpc", "ec2-instance", "rds")	Easier to reuse and reason about
Always define variables.tf and outputs.tf explicitly	Makes the module's interface clear
Use sensible defaults for optional variables	Reduces boilerplate for callers
Avoid hardcoding provider blocks inside child modules	Providers should be configured in the root module
Version-pin modules from Git/Registry (?ref=v1.2.0)	Prevents unexpected breaking changes
Add a README.md documenting inputs/outputs	Helps others (and future you) use it correctly
Typical real-world layout

project/
├── main.tf              # calls modules
├── variables.tf
├── outputs.tf
├── providers.tf
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── ec2-instance/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── rds/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
