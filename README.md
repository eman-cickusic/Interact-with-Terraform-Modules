# Interacting with Terraform Modules

This repository demonstrates how to use and create Terraform modules for infrastructure management. It includes examples of using existing modules from the Terraform Registry and creating custom modules.

## Table of Contents
- [Overview](#overview)
- [What are Modules For?](#what-are-modules-for)
- [Task 1: Using Modules from the Registry](#task-1-using-modules-from-the-registry)
  - [Setup and Requirements](#setup-and-requirements)
  - [Using the Network Module](#using-the-network-module)
  - [Understanding Module Inputs and Outputs](#understanding-module-inputs-and-outputs)
- [Task 2: Building a Module](#task-2-building-a-module)
  - [Module Structure](#module-structure)
  - [Creating a GCS Static Website Module](#creating-a-gcs-static-website-module)
  - [Using Your Custom Module](#using-your-custom-module)
  - [Uploading Files to the Bucket](#uploading-files-to-the-bucket)

## Overview

As you manage your infrastructure with Terraform, increasingly complex configurations will be created. There is no intrinsic limit to the complexity of a single Terraform configuration file or directory, so it is possible to continue writing and updating your configuration files in a single directory. However, if you do, you may encounter one or more of the following problems:

- Understanding and navigating the configuration files will become increasingly difficult.
- Updating the configuration will become more risky, because an update to one block may cause unintended consequences to other blocks of your configuration.
- Duplication of similar blocks of configuration may increase, for example, when you configure separate dev/staging/production environments, which will cause an increasing burden when updating those parts of your configuration.
- If you want to share parts of your configuration between projects and teams, cutting and pasting blocks of configuration between projects could be error-prone and hard to maintain.

## What are Modules For?

Here are some of the ways that modules help solve the problems listed above:

- **Organize configuration**: Modules make it easier to navigate, understand, and update your configuration by keeping related parts of your configuration together. Even moderately complex infrastructure can require hundreds or thousands of lines of configuration to implement. By using modules, you can organize your configuration into logical components.

- **Encapsulate configuration**: Another benefit of using modules is to encapsulate configuration into distinct logical components. Encapsulation can help prevent unintended consequences—such as a change to one part of your configuration accidentally causing changes to other infrastructure—and reduce the chances of simple errors like using the same name for two different resources.

- **Re-use configuration**: Writing all of your configuration without using existing code can be time-consuming and error-prone. Using modules can save time and reduce costly errors by re-using configuration written either by yourself, other members of your team, or other Terraform practitioners who have published modules for you to use. You can also share modules that you have written with your team or the general public, giving them the benefit of your hard work.

- **Provide consistency and ensure best practices**: Modules also help to provide consistency in your configurations. Consistency makes complex configurations easier to understand, and it also helps to ensure that best practices are applied across all of your configuration. For example, cloud providers offer many options for configuring object storage services, such as Amazon S3 (Simple Storage Service) or Google's Cloud Storage buckets. Many high-profile security incidents have involved incorrectly secured object storage, and because of the number of complex configuration options involved, it's easy to accidentally misconfigure these services.

## Task 1: Using Modules from the Registry

### Setup and Requirements

To use modules from the Terraform Registry:

1. Clone the example project from the Google Terraform modules GitHub repository:
```bash
git clone https://github.com/terraform-google-modules/terraform-google-network
cd terraform-google-network
git checkout tags/v6.0.1 -b v6.0.1
```

2. Navigate to the `examples/simple_project` directory to examine the example configuration.

### Using the Network Module

The main configuration file (`main.tf`) references the Google Network module from the Terraform Registry:

```hcl
module "test-vpc-module" {
  source       = "terraform-google-modules/network/google"
  version      = "~> 6.0"
  project_id   = var.project_id 
  network_name = var.network_name
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "REGION"
    },
    {
      subnet_name           = "subnet-02"
      subnet_ip             = "10.10.20.0/24"
      subnet_region         = "REGION"
      subnet_private_access = "true"
      subnet_flow_logs      = "true"
    },
    {
      subnet_name               = "subnet-03"
      subnet_ip                 = "10.10.30.0/24"
      subnet_region             = "REGION"
      subnet_flow_logs          = "true"
      subnet_flow_logs_interval = "INTERVAL_10_MIN"
      subnet_flow_logs_sampling = 0.7
      subnet_flow_logs_metadata = "INCLUDE_ALL_METADATA"
      subnet_flow_logs_filter   = "false"
    }
  ]
}
```

### Understanding Module Inputs and Outputs

The required inputs for this module are:
- `network_name`: The name of the network being created
- `project_id`: The ID of the project where this VPC will be created
- `subnets`: The list of subnets being created

Variables are defined in `variables.tf`:

```hcl
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "YOUR_PROJECT_ID"
}

variable "network_name" {
  description = "The name of the VPC network being created"
  default     = "example-vpc"
}
```

Outputs are defined in `outputs.tf` to access values from the module:

```hcl
output "network_name" {
  value       = module.test-vpc-module.network_name
  description = "The name of the VPC being created"
}

output "network_self_link" {
  value       = module.test-vpc-module.network_self_link
  description = "The URI of the VPC being created"
}

# Additional outputs...
```

## Task 2: Building a Module

### Module Structure

A typical file structure for a Terraform module is:

```
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

Each file serves a purpose:
- `LICENSE`: Contains the license under which your module will be distributed
- `README.md`: Contains documentation in markdown format that describes how to use your module
- `main.tf`: Contains the main set of configurations for your module
- `variables.tf`: Contains the variable definitions for your module
- `outputs.tf`: Contains the output definitions for your module

### Creating a GCS Static Website Module

To create a module for Google Cloud Storage static website hosting:

1. Create the directory structure:
```bash
mkdir -p modules/gcs-static-website-bucket
cd modules/gcs-static-website-bucket
touch website.tf variables.tf outputs.tf README.md LICENSE
```

2. Create the module configuration files:

**website.tf**:
```hcl
resource "google_storage_bucket" "bucket" {
  name               = var.name
  project            = var.project_id
  location           = var.location
  storage_class      = var.storage_class
  labels             = var.labels
  force_destroy      = var.force_destroy
  uniform_bucket_level_access = true

  versioning {
    enabled = var.versioning
  }

  dynamic "retention_policy" {
    for_each = var.retention_policy == null ? [] : [var.retention_policy]
    content {
      is_locked        = var.retention_policy.is_locked
      retention_period = var.retention_policy.retention_period
    }
  }

  dynamic "encryption" {
    for_each = var.encryption == null ? [] : [var.encryption]
    content {
      default_kms_key_name = var.encryption.default_kms_key_name
    }
  }

  dynamic "lifecycle_rule" {
    for_each = var.lifecycle_rules
    content {
      action {
        type          = lifecycle_rule.value.action.type
        storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
      }
      condition {
        age                   = lookup(lifecycle_rule.value.condition, "age", null)
        created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
        with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
        matches_storage_class = lookup(lifecycle_rule.value.condition, "matches_storage_class", null)
        num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
      }
    }
  }
}
```

3. Define variables and outputs for your module:

**variables.tf**:
```hcl
variable "name" {
  description = "The name of the bucket."
  type        = string
}

variable "project_id" {
  description = "The ID of the project to create the bucket in."
  type        = string
}

variable "location" {
  description = "The location of the bucket."
  type        = string
}

# Additional variables...
```

**outputs.tf**:
```hcl
output "bucket" {
  description = "The created storage bucket"
  value       = google_storage_bucket.bucket
}
```

### Using Your Custom Module

In your root directory, reference your custom module:

```hcl
module "gcs-static-website-bucket" {
  source = "./modules/gcs-static-website-bucket"

  name       = var.name
  project_id = var.project_id
  location   = "REGION"

  lifecycle_rules = [{
    action = {
      type = "Delete"
    }
    condition = {
      age        = 365
      with_state = "ANY"
    }
  }]
}
```

### Uploading Files to the Bucket

After creating the bucket with your module, you can upload files:

```bash
# Download sample content
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/master/modules/aws-s3-static-website-bucket/www/index.html > index.html
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/blob/master/modules/aws-s3-static-website-bucket/www/error.html > error.html

# Upload to bucket
gsutil cp *.html gs://YOUR-BUCKET-NAME
```

You can then view your website at: `https://storage.cloud.google.com/YOUR-BUCKET-NAME/index.html`

## Cleanup

To destroy the infrastructure you created:

```bash
terraform destroy
```

After responding to the prompt with `yes`, Terraform will destroy all of the resources created.
