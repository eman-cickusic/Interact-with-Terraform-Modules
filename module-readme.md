# GCS static website bucket

This module provisions Cloud Storage buckets configured for static website hosting.

## Usage

```hcl
module "static_website" {
  source = "./modules/gcs-static-website-bucket"

  name       = "my-static-website"
  project_id = "my-project-id"
  location   = "us-central1"

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

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| name | The name of the bucket | `string` | n/a | yes |
| project_id | The ID of the project to create the bucket in | `string` | n/a | yes |
| location | The location of the bucket | `string` | n/a | yes |
| storage_class | The Storage Class of the new bucket | `string` | `null` | no |
| labels | A set of key/value label pairs to assign to the bucket | `map(string)` | `null` | no |
| bucket_policy_only | Enables Bucket Policy Only access to a bucket | `bool` | `true` | no |
| versioning | While set to true, versioning is fully enabled for this bucket | `bool` | `true` | no |
| force_destroy | When deleting a bucket, this boolean option will delete all contained objects | `bool` | `true` | no |
| lifecycle_rules | The bucket's Lifecycle Rules configuration | `list(object)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| bucket | The created storage bucket |
