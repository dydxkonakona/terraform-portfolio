# Launch Amazon S3 using Terraform
```terraform
provider "aws" {
  region = "ap-southeast-1"
}

# Bucket creation
resource "aws_s3_bucket" "sample" {
  bucket = "mysampleterraformbucket"
}

# Allow public access settings
resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket = aws_s3_bucket.sample.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# Enable static website hosting
resource "aws_s3_bucket_website_configuration" "website" {
  bucket = aws_s3_bucket.sample.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }
}

# Public read-only bucket policy
data "aws_iam_policy_document" "public_read" {
  statement {
    principals {
      type        = "*"
      identifiers = ["*"]
    }

    effect = "Allow"

    actions = [
      "s3:GetObject",
    ]

    resources = [
      "${aws_s3_bucket.sample.arn}/*"
    ]
  }
}

resource "aws_s3_bucket_policy" "allow_access_anywhere" {
  bucket = aws_s3_bucket.sample.id
  policy = data.aws_iam_policy_document.public_read.json
}

# Upload files from local "upload" folder
resource "aws_s3_object" "website_files" {
  for_each = fileset("${path.module}/upload", "*")

  bucket = aws_s3_bucket.sample.id
  key    = each.value
  source = "${path.module}/upload/${each.value}"

  etag         = filemd5("${path.module}/upload/${each.value}")
  content_type = lookup(
    {
      html = "text/html"
      css  = "text/css"
      js   = "application/javascript"
      png  = "image/png"
      jpg  = "image/jpeg"
      jpeg = "image/jpeg"
      gif  = "image/gif"
    },
    lower(element(split(".", each.value), length(split(".", each.value)) - 1)),
    "binary/octet-stream"
  )
}

# Output the website endpoint
output "website_endpoint" {
  value = aws_s3_bucket_website_configuration.website.website_endpoint
}

```

Interestingly I find the when using bucket naming scheme with *terraform-\** the bucket website endpoint opens briefly and just closes so I never get to see the website.
