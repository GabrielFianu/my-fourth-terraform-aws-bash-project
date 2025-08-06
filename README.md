# EC2 + S3 + CloudWatch Monitoring (Terraform + Bash)

This project provisions:
- A secure EC2 Ubuntu server with Nginx pre-installed via Bash
- An encrypted, versioned S3 bucket for backups
- CloudWatch monitoring agent for CPU and memory metrics

## 🛠 Technologies
- Terraform
- Bash
- AWS EC2, S3, CloudWatch
- Ubuntu

## 🔧 How to Use
```bash
terraform init
terraform apply -auto-approve

## Notes
- All resources are Free Tier eligible.
- Tags help track and manage infrastructure.
