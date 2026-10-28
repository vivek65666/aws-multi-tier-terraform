# Multi-Tier Architecture on AWS using Terraform

Terraform code to provision a three-tier architecture (web, application and database) on AWS, with load balancing and auto scaling on the web and application tiers.

## Architecture

```
Internet
   |
[ Web ALB ]  (public subnets)
   |
[ Web Auto Scaling Group ]  (web subnets)
   |
[ App ALB ]  (internal)
   |
[ App Auto Scaling Group ]  (private app subnets)
   |
[ RDS Database ]  (private DB subnets)
```

<!-- Add your architecture diagram here: ![Architecture](images/architecture.png) -->

| Tier | Components |
|------|------------|
| Web | Application Load Balancer, target group, launch template, Auto Scaling group |
| Application | Application Load Balancer, target group, launch template, Auto Scaling group |
| Database | Amazon RDS instance in a private DB subnet group |
| Networking | VPC, public and private subnets, Internet Gateway, NAT Gateway, Elastic IP, route tables |
| Security | Separate security groups for each ALB, each Auto Scaling group and the database |

## Security Design

- Only the web load balancer accepts traffic from the internet.
- Each tier accepts traffic only from the tier in front of it, enforced with security groups.
- The database sits in private subnets and is reachable only from the application tier.
- Private instances reach the internet through the NAT Gateway.

## Project Structure

| File | Purpose |
|------|---------|
| `provider.tf` | AWS provider configuration |
| `backend.tf` | Terraform state backend |
| `variables.tf` | Input variables |
| `output.tf` | Output values |
| `vpc.tf` | VPC |
| `web-subnets.tf`, `app-subnets.tf`, `db-subnets.tf` | Subnets for each tier |
| `internet-gateway.tf`, `nat-gw.tf`, `eip.tf` | Internet and NAT connectivity |
| `public-route-table.tf`, `private-route-table.tf` | Routing |
| `alb-web.tf`, `alb-app.tf` | Load balancers |
| `tg-web.tf`, `tg-app.tf` | Target groups |
| `launch-template-web.tf`, `launch-template-app.tf` | Launch templates |
| `asg-web.tf`, `asg-app.tf` | Auto Scaling groups |
| `alb-web-sg.tf`, `alb-app-sg.tf`, `asg-web-sg.tf`, `asg-app-sg.tf`, `db-sg.tf` | Security groups |
| `db-subnet-group.tf`, `rds.tf` | Database |
| `user-data.sh` | Instance bootstrap script |

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) installed
- An AWS account with IAM credentials configured (`aws configure`)
- Git

## Deployment

```bash
git clone https://github.com/vivek65666/aws-multi-tier-terraform.git
cd aws-multi-tier-terraform

terraform init
terraform plan
terraform apply
```

Review the plan before confirming. Update `variables.tf` (region, instance type, database settings) to suit your needs. Do not commit passwords or secrets. Pass them through environment variables or a `terraform.tfvars` file that is listed in `.gitignore`.

## Cleanup

```bash
terraform destroy
```

## Cost Warning

The NAT Gateway, load balancers and RDS instance are billed while they run. Run `terraform destroy` when you finish testing.

## What I Learned

- Designing a VPC with separate public and private subnets
- Placing load balancers and Auto Scaling groups across tiers
- Controlling traffic between tiers with security groups
- Splitting Terraform code into files and using variables and outputs

## License

MIT
