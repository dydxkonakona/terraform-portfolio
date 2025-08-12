# Launch Amazon S3 using Terraform
```terraform
provider "aws" {
  region = "ap-southeast-1"
}

# Create the bucket
resource "aws_s3_bucket" "sample" {
  bucket = "sample-tf-bucket-87483"

  # example tags as additional info on the bucket
  tags = {
    Key = "Department",
    Value = "Marketing"
  }
}

# Disable all S3 bucket-level Public Access Block
resource "aws_s3_bucket_public_access_block" "sample" {
  bucket = aws_s3_bucket.sample.id

  block_public_acls = false
  block_public_policy = false
  ignore_public_acls = false
  restrict_public_buckets = false
}

# Enable website hosting

resource "aws_s3_bucket_website_configuration" "sample" {
  bucket = aws_s3_bucket.sample.id

  index_document {
    suffix = "index.html"
  }
}

# Assign bucket policy to allow access from anywhere
resource "aws_s3_bucket_policy" "allow_access_from_anywhere" {
  bucket = aws_s3_bucket.sample.id
  policy = data.aws_iam_policy_document.allow_access_from_anywhere.json
}

# Create the bucket policy here in the script
data "aws_iam_policy_document" "allow_access_from_anywhere" {
  statement {
    principals {
      type = "*"
      identifiers = ["*"]
    }
    effect = "Allow"
    actions = [
      "s3:GetObject",
    ]
    resources = [
      "${aws_s3_bucket.sample.arn}/*",
    ]
  }

  statement {
    principals {
      type = "*"
      identifiers = ["*"]
    }
    effect = "Deny"
    actions = [
      "s3:DeleteObject"
    ]
    resources = [
      "${aws_s3_bucket.sample.arn}/index.html",
      "${aws_s3_bucket.sample.arn}/style.html",
    ]
  }
}

# Upload files on the bucket
resource "aws_s3_object" "object" {
  for_each = fileset("./upload", "*")
  bucket = aws_s3_bucket.sample.id
  key = each.value
  source = "./upload/${each.value}"
  # etag makes the file update when it changes; see https://stackoverflow.com/questions/56107258/terraform-upload-file-to-s3-on-every-apply
  etag   = filemd5("./upload/${each.value}")
}
```
