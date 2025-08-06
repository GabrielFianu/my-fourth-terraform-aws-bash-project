# Provisioning of EC2, S3 and CloudWatch Monitoring (Terraform + Bash)

## Introduction
Goal is to provision a secure EC2 instance, create an S3 bucket for backups, and configure CloudWatch monitoring using Terraform and a Bash script.

**Project Structure**

ec2-s3-cloudwatch/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── user-data.sh
├── cloudwatch-agent-config.json
└── README.md


### 1. Creating Terraform configuration files
**Note** create directory and change to that directory.
``` mkdir name of directory
    cd name of directory
```

a. Create **provider.tf** file and the following below: 

```
provider "aws" {
  region = var.aws_region
}
```


<img width="692" height="185" alt="Image" src="https://github.com/user-attachments/assets/6c47b28b-c56f-4325-bf53-55f78cfadd18" />


b. Create **main.tf** file and the following below: 

```
resource "aws_s3_bucket" "backup" {
  bucket = var.bucket_name
}

resource "aws_s3_bucket_versioning" "backup_versioning" {
  bucket = aws_s3_bucket.backup.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "backup_encryption" {
  bucket = aws_s3_bucket.backup.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_security_group" "web_sg" {
  name        = "web_sg"
  description = "Allow HTTP and SSH"
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0fc5d935ebf8bc3bc" # Ubuntu 22.04 in us-east-1
  instance_type = var.instance_type
  key_name      = "actionman" # Replace with your key name
  user_data     = file("user-data.sh")
  security_groups = [aws_security_group.web_sg.name]

  tags = {
    Name = "WebServer"
  }
}
```

<img width="716" height="707" alt="Image" src="https://github.com/user-attachments/assets/bc0c8025-d12e-4e9f-a7d5-929058bb3a57" />

c. Create and configure variables.tf file
```
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.micro"
}

variable "bucket_name" {
  default = "my-secure-backup-bucket-123456"
}
```

<img width="662" height="328" alt="Image" src="https://github.com/user-attachments/assets/a92c1e3a-bf07-4522-89d4-1084c887199c" />


d. Create and Configure Outputs.tf file
```
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

output "s3_bucket_name" {
  value = aws_s3_bucket.backup.bucket
}
```
<img width="720" height="293" alt="Image" src="https://github.com/user-attachments/assets/1f6b837d-1fe8-437c-aecc-b2f0191d621c" />

e. Create and Configure user-data.sh file
```
#!/bin/bash
apt update -y
apt install nginx -y
systemctl enable nginx
systemctl start nginx

# Install CloudWatch Agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
dpkg -i amazon-cloudwatch-agent.deb
cp /opt/aws/amazon-cloudwatch-agent/bin/config.json /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```


<img width="722" height="373" alt="Image" src="https://github.com/user-attachments/assets/706d2d0e-f01f-4c9b-a6a9-54974f73ca0a" />

f. cloudwatch-agent-config.json
(Optional if you want custom metrics)

```
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"]
      },
      "cpu": {
        "measurement": ["cpu_usage_idle"],
        "totalcpu": true
      }
    }
  }
}
```

<img width="683" height="487" alt="Image" src="https://github.com/user-attachments/assets/6f34d503-81cb-4083-b16b-5c69e7b54709" />


---

### 2. Initialize Terraform
Run

```
terraform init
```

<img width="578" height="709" alt="Image" src="https://github.com/user-attachments/assets/15d26568-8d68-44cd-83c2-a673adcfb9bb" />

---

### 3. Validate & Plan

Run

```
terraform validate
```

<img width="694" height="98" alt="Image" src="https://github.com/user-attachments/assets/9522242d-2f45-448e-a672-ab03e4621d41" />

Run

```
terraform plan
```

<img width="851" height="709" alt="Image" src="https://github.com/user-attachments/assets/bb504a8a-da01-4512-8a46-10636b232533" />


---

### 4. Deploy Resources


```
terraform apply
```
Type and enter **yes** when prompted.


<img width="849" height="705" alt="Image" src="https://github.com/user-attachments/assets/b31546b0-46a7-4f98-853b-b10da92ecb5d" />

---

### 5. Visit the Public IP
Terraform will print something like:

Outputs:
public_ip = "IP address"

**Test It**
After running terraform apply, copy the output public IP and open it in your browser. You should see the Nginx welcome page.

<img width="815" height="136" alt="Image" src="https://github.com/user-attachments/assets/f5a1d591-315d-4aeb-971c-2ef0f3c32f0f" />


---

###. 6. Confirm resources provisioned on terraform

Run

```
terraform state list
```
and
```
terraform show
```

<img width="630" height="168" alt="Image" src="https://github.com/user-attachments/assets/b262a99a-a5fa-4250-9426-12caa072ddc9" />

<img width="853" height="213" alt="Image" src="https://github.com/user-attachments/assets/a9ca3f32-543d-41ec-85b9-15c9f7c821c0" />

---

### 7. Cleanup (Avoid Charges)
a. To destroy all provisioned infrastructure:

Run
```
terraform destroy
```

b. Run the following to confirm
bash
```
terraform state list
```
and
```
terraform show
```

---

###. 6. Confirm resources provisioned and destroyed from the Console

![Image](https://github.com/user-attachments/assets/c8c03cef-adc6-4489-b075-4ef044e4c2af)


![Image](https://github.com/user-attachments/assets/a75204dd-4949-4eb5-aaed-dc1d2520a954)


![Image](https://github.com/user-attachments/assets/ebb2f240-4ba7-40b2-bc7c-176f7214a4ab)


