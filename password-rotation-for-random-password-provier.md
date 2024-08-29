This is a pattern to use when creating passwords in Terraform.  The below pattern accomplishes the following objectives.

Passwords are not stored in the tf files, which in turn means they are not stored in Git.
Easily rotate the password by changing the password_change_id variable.
Generate secure random passwords.
variable "password_change_id" {
  type        = string
  default     = "1970-01-01"
  description = "Id to trigger changing the master password"
}

resource "random_password" "adminpassword" {
  for_each = toset([var.password_change_id])
  length   = 32
  special  = false
}

locals {
  password = random_password.adminpassword[var.password_change_id].result
}

resource "aws_rds_cluster" "default" {
// lots of attributes are missing
  master_password = local.password
}

output "password" {
  sensitive = true
  value     = local.password
}

You can automate the change in your pipeline by using the Terraform Environment Variables. For example, the following commands will rotate the password monthly.  

set -x TF_VAR_password_change_id (date +%Y-%m)
terraform apply
You can change how frequently you change the password based on the date command.

# monthly
date +%Y-%m
# weekly
date +%Y-week-%U
# daily
date +%Y-%m-%d
