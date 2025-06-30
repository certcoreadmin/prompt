## Role & Expertise
You are a Terraform expert with 20+ years of experience designing, writing, and reviewing Terraform code for production-grade infrastructure. You know all major providers, with a specialization in GCP, Terraform best practices, module design, and security standards. You write clean, commented, and production-ready code.

---

## Objective
Generate a complete and ready-to-apply set of Terraform files that provision the GCP infrastructure defined below, adhering to all specified best practices.

---

## 1. Context & Scope
- **Cloud Provider(s):** `google`
- **Region(s):** `<e.g. us-central1, europe-west2>`
- **Project ID:** `<your-gcp-project-id>`
- **Environment:** `<e.g. dev, staging, prod>`
- **Terraform Version:** `~> 1.5`
- **Provider Versions:** `hashicorp/google ~> 5.0`
- **Intended Use:** `<e.g. multi-tier web application, data processing pipeline, core networking>`

---

## 2. Resources to Create
List each resource and its key properties.

| Resource Type | Name/Identifier | Key Properties |
|---|---|---|
| `google_compute_network` | `project_vpc` | `auto_create_subnetworks=false` |
| `google_compute_subnetwork` | `project_subnet` | `ip_cidr_range`, `region` |
| `google_compute_instance` | `web_server_vm` | `machine_type`, `boot_disk.initialize_params.image`, `network_interface`, `tags` |
| `google_sql_database_instance` | `db_instance` | `database_version`, `settings.tier`, `deletion_protection` |
| `google_compute_global_address` | `private_service_access_range` | `purpose=VPC_PEERING`, `address_type=INTERNAL`, `prefix_length=16` |
| `google_service_networking_connection` | `private_vpc_connection` | Connects the VPC to Google services for Cloud SQL. `network`, `service`, `reserved_peering_ranges` |
| *(Add or remove rows as needed)* | | |

---

## 2.1. Firewall Rules
Define specific ingress rules for VPC firewall.

| Firewall Rule Name | Direction | Action | Protocol & Ports | Target Tags | Source Ranges/Tags | Description |
|---|---|---|---|---|---|---|
| `allow-ssh` | INGRESS | ALLOW | `tcp:22` | `["web-server"]` | `["0.0.0.0/0"]` | Allow inbound SSH traffic from anywhere (for dev) |
| `allow-https` | INGRESS | ALLOW | `tcp:443` | `["web-server"]` | `["0.0.0.0/0"]` | Allow inbound HTTPS traffic from anywhere |
| `allow-internal-db` | INGRESS | ALLOW | `tcp:3306` | `["sql-database"]` | `["web-server"]` | Allow DB access from instances with the web-server tag |
| *(Add or remove rows as needed)* | | | | | | |

---

## 2.2. Data Sources to Use (Optional)
Specify any data sources needed to fetch information.

| Data Source Type | Name/Identifier | Purpose & Key Filters |
|---|---|---|
| `google_compute_image` | `latest_debian` | To fetch the latest Debian image for boot disks. `family="debian-11"`, `project="debian-cloud"` |
| *(Add or remove rows as needed)* | | |

---

## 3. Inputs & Variables
Define all external parameters as Terraform variables.

```hcl
variable "project_id" {
  description = "The GCP project ID to deploy resources in."
  type        = string
}

variable "project_name" {
  description = "The name of the project, used for labeling resources."
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

variable "gcp_region" {
  description = "The GCP region to deploy resources in."
  type        = string
  default     = "us-central1"
}

# ...add any other variables as needed
```

---

## 4. Outputs
Specify values to expose after a successful `terraform apply`.

```hcl
output "vpc_network_name" {
  description = "The name of the created VPC network."
  value       = google_compute_network.project_vpc.name
}

output "web_server_external_ip" {
  description = "External IP address of the web server VM."
  value       = google_compute_instance.web_server_vm.network_interface[0].access_config[0].nat_ip
}

output "db_instance_connection_name" {
  description = "The connection name for the Cloud SQL database instance."
  value       = google_sql_database_instance.db_instance.connection_name
  sensitive   = true
}

output "db_instance_private_ip" {
  description = "The private IP address of the Cloud SQL instance."
  value       = google_sql_database_instance.db_instance.private_ip_address
  sensitive   = true
}

# ...add any other useful outputs
```

---

## 5. Module Structure (Optional)
If you want the code organized into reusable modules, specify the structure.

- **`modules/vpc/`** → VPC network, subnets, firewall rules, PSA
- **`modules/gce/`** → Compute instances, instance templates
- **`modules/cloud-sql/`** → Cloud SQL instances, users, databases

---

## 6. Best Practices & Constraints
- **State Management:** Configure a remote backend using a Google Cloud Storage (GCS) bucket.
- **Labeling:** All resources that support it MUST have `environment` and `project` labels.
- **Security:** Follow the principle of least privilege for all GCP IAM roles and Firewall Rules. Do not use hardcoded secrets.
- **Readability:** Add comments to explain complex logic or non-obvious configurations.
- **Dependencies:** Use explicit `depends_on` only when absolutely necessary; prefer implicit dependency resolution through resource interpolation.

---

## Task
Based on all the requirements above, generate the following files:
1.  **`providers.tf`**: Configures the Google provider and required versions, including the GCS remote state backend.
2.  **`variables.tf`**: Contains all variable definitions.
3.  **`main.tf`**: Defines the core resources (or calls modules if specified). **This file must start with a comment block describing its purpose and the infrastructure it creates.**
4.  **`outputs.tf`**: Exposes the key attributes of the created infrastructure.
5.  **`terraform.tfvars.example`**: Provides a sample file demonstrating how to set the required variable values.

Ensure the final code is complete, correct, and ready for `terraform init` and `terraform apply`.
