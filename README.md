# terraform-cloudfront
Using Terraform to create a CloudFront Application in AWS


## To use Terraform to create an Amazon CloudFront distribution

To use Terraform to create an Amazon CloudFront distribution, follow these steps:

1. **Install Terraform**:
   Ensure that Terraform is installed on your system. You can download it from the [Terraform website](https://www.terraform.io/downloads.html).

2. **Set Up AWS Credentials**:
   Make sure your AWS credentials are set up on your machine. You can use the AWS CLI to configure them:
   ```sh
   aws configure
   ```

3. **Create a Terraform Configuration File**:
   Create a new directory for your Terraform project and navigate into it. Then, create a file named `main.tf`.

4. **Write Terraform Configuration**:
   Add the necessary Terraform configuration to `main.tf` to create a CloudFront distribution. Here is an example configuration:

   ```hcl
   provider "aws" {
     region = "us-east-1"  # Change to your preferred region
   }

   resource "aws_s3_bucket" "bucket" {
     bucket = "my-cloudfront-bucket"  # Change to your preferred bucket name
     acl    = "public-read"

     website {
       index_document = "index.html"
       error_document = "error.html"
     }
   }

   resource "aws_cloudfront_origin_access_identity" "origin_access_identity" {
     comment = "My CloudFront Origin Access Identity"
   }

   resource "aws_cloudfront_distribution" "distribution" {
     origin {
       domain_name = aws_s3_bucket.bucket.bucket_regional_domain_name
       origin_id   = "S3-my-cloudfront-bucket"

       s3_origin_config {
         origin_access_identity = aws_cloudfront_origin_access_identity.origin_access_identity.cloudfront_access_identity_path
       }
     }

     enabled             = true
     is_ipv6_enabled     = true
     comment             = "My CloudFront Distribution"
     default_root_object = "index.html"

     default_cache_behavior {
       allowed_methods  = ["GET", "HEAD"]
       cached_methods   = ["GET", "HEAD"]
       target_origin_id = "S3-my-cloudfront-bucket"

       forwarded_values {
         query_string = false
         cookies {
           forward = "none"
         }
       }

       viewer_protocol_policy = "redirect-to-https"
       min_ttl                = 0
       default_ttl            = 3600
       max_ttl                = 86400
     }

     restrictions {
       geo_restriction {
         restriction_type = "none"
       }
     }

     viewer_certificate {
       cloudfront_default_certificate = true
     }
   }
   ```

5. **Initialize Terraform**:
   Initialize the Terraform project. This will download the necessary provider plugins.
   ```sh
   terraform init
   ```

6. **Plan and Apply**:
   Run the `terraform plan` command to see the changes that will be made. Then, run `terraform apply` to create the CloudFront distribution and related resources.
   ```sh
   terraform plan
   terraform apply
   ```

Here is a breakdown of the key components in the configuration:

- `provider "aws"`: Specifies the AWS provider and region.
- `resource "aws_s3_bucket"`: Creates an S3 bucket that will be used as the CloudFront origin.
- `resource "aws_cloudfront_origin_access_identity"`: Creates an Origin Access Identity (OAI) to restrict access to the S3 bucket through CloudFront.
- `resource "aws_cloudfront_distribution"`: Creates the CloudFront distribution.

Modify the configuration as needed to fit your specific requirements. For instance, you can customize cache behaviors, security settings, and more.