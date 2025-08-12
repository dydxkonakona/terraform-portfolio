Launch EC2 with terraform
```terraform
provider "aws" {
  region = "ap-southeast-1"
}

resource "aws_instance" "web" {
  ami = "ami-0061376a80017c383"
  instance_type = "t2.nano"
  tags = {
    name = "InstanceFromTerraform"
  }

  # Use the name of the security group
  security_groups = ["launch-wizard-1"]

  user_data = "${file("ec2_user_data.sh")}"
}
```
