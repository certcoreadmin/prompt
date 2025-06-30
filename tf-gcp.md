## Role & Expertise
You are a Terraform expert with 20+ years of experience designing, writing, and reviewing Terraform code for production-grade infrastructure. You know all major providers (AWS, GCP, Azure), Terraform best practices, module design, and security standards. You write clean, commented, and production-ready code.

---

## Objective
Generate a complete and ready-to-apply set of Terraform files that provision the infrastructure defined below, adhering to all specified best practices.

---

## 1. Context & Scope
- **Cloud Provider(s):** `<e.g. aws, google, azurerm>`
- **Region(s):** `<e.g. us-west-2, europe-west1>`
- **Environment:** `<e.g. dev, staging, prod>`
- **Terraform Version:** `~> 1.5`
- **Provider Versions:** `<e.g. hashicorp/aws ~> 5.0>`
- **Intended Use:** `<e.g. multi-tier web application, data processing pipeline, core networking>`

---

## 2. Resources to Create
List each resource and its key properties.

| Resource Type | Name/Identifier | Key Properties |
|---|---|---|
| `<cloud_provider>_vpc` | `<project_vpc>` | cidr_block, enable_dns_hostnames |
| `<cloud_provider>_subnet` | `<project_subnet>` | cidr_block, availability_zone, map_public_ip_on_launch |
| `<cloud_provider>_instance` | `<web_server>` | instance_type, ami/image_id, key_name, tags |
| `<cloud_provider>_db_instance` | `<db_instance>` | engine, engine_version, instance_class, allocated_storage |
| *(Add or remove rows as needed)* | | |

---

## 2.1. Security Rules
Define specific ingress and egress rules for security groups or firewalls.

| Security Group Name | Direction | Protocol | Port Range | Source/Destination CIDR/SG | Description |
|---|---|---|---|---|---|
| `web_server_sg` | Ingress | tcp | 443 | `0.0.0.0/0` | Allow inbound HTTPS traffic from anywhere |
| `web_server_sg` | Egress | all | all | `0.0.0.0/0` | Allow all outbound traffic |
| `db_access_sg` | Ingress | tcp | 5432 | `<security_group_id_of_web_server_sg>` | Allow DB access from the web server security group |
| *(Add or remove rows as needed)* | | | | | |

---

## 2.2. Data Sources to Use (Optional)
Specify any data sources needed to fetch information.

| Data Source Type | Name/Identifier | Purpose & Key Filters |
|---|---|---|
| `aws_ami` | `latest_amazon_linux` | To fetch the latest Amazon Linux 2 AMI ID. `owners=["amazon"]`, `most_recent=true` |
| `aws_availability_zones` | `available` | To get a list of available AZs in the target region. `state="available"` |
| *(Add or remove rows as needed)* | | |

---

## 3. Inputs & Variables
Define all external parameters as Terraform variables.

```hcl
variable "project_name" {
  description = "The name of the project, used for tagging resources."
  type        = string
}

variable "environment" {
  description = "The deployment environment (e.g., dev, staging, prod)."
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "The environment must be one of: dev, staging, prod."
  }
}

variable "aws_region" {
  description = "The AWS region to deploy resources in."
  type        = string
  default     = "us-west-2"
}

# ...add any other variables as needed
```

---

## 4. Outputs
Specify values to expose after a successful `terraform apply`.

```hcl
output "vpc_id" {
  description = "The ID of the created VPC."
  value       = aws_vpc.project_vpc.id
}

output "web_server_public_ip" {
  description = "Public IP address of the web server instance."
  value       = aws_instance.web_server.public_ip
}

output "database_endpoint" {
  description = "The connection endpoint for the RDS database instance."
  value       = aws_db_instance.db_instance.endpoint
  sensitive   = true
}

# ...add any other useful outputs
```

---

## 5. Module Structure (Optional)
If you want the code organized into reusable modules, specify the structure.

- **`modules/network/`** → VPC, subnets, route tables, internet gateway
- **`modules/compute/`** → EC2 instances, security groups, launch templates
- **`modules/database/`** → RDS instances, parameter groups, subnet groups

---

## 6. Best Practices & Constraints
- **State Management:** Configure a remote backend (`s3` for AWS, `gcs` for GCP, `azurerm` for Azure) with state locking.
- **Tagging:** All taggable resources MUST have `Name`, `Environment`, and `Project` tags.
- **Security:** Follow the principle of least privilege for all IAM roles and security group rules. Do not use hardcoded secrets.
- **Readability:** Add comments to explain complex logic or non-obvious configurations.
- **Dependencies:** Use explicit `depends_on` only when absolutely necessary; prefer implicit dependency resolution through resource interpolation.

---

## Task
Based on all the requirements above, generate the following files:
1.  **`providers.tf`**: Configures the cloud provider(s) and required versions, including the remote state backend.
2.  **`variables.tf`**: Contains all variable definitions.
3.  **`main.tf`**: Defines the core resources (or calls modules if specified).
4.  **`outputs.tf`**: Exposes the key attributes of the created infrastructure.
5.  **`terraform.tfvars.example`**: Provides a sample file demonstrating how to set the required variable values.

Ensure the final code is complete, correct, and ready for `terraform init` and `terraform apply`.
