[![CI](https://github.com/MohamedAboMousallam/node-hello/actions/workflows/ci.yml/badge.svg)](https://github.com/MohamedAboMousallam/node-hello/actions/workflows/ci.yml)

## Assumptions

Before implementing the solution, I made a few assumptions to guide my submission in alignment with best practices:

1. **Code Formatting & Quality**
   - A `.prettierrc` file was found in the repository, suggesting that consistent formatting was expected.
   - I added both **Prettier** and **ESLint** to ensure automatic code formatting and linting via GitHub Actions.

2. **Log Aggregation with New Relic**
   - Two approaches were considered (from the new-relic docs):
     - **S3 Bucket + Lambda Forwarder**
     - **Sidecar Container (FireLens)**
   - I selected the **FireLens Sidecar** approach for this assignment due to its real-time log shipping and direct integration with ECS.
3. **Secret Management**
   - Instead of hardcoding secrets (e.g., New Relic API key), I used **AWS SSM Parameter Store**.
   - The license key was securely fetched by the ECS task at runtime using `secretOptions` but it was commented because later I changed from JSON fomat to plain text (suggested by new-relic docs).

---

## 🛠️ Setup and Usage

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [Terraform](https://developer.hashicorp.com/terraform/downloads)
- AWS account with access to:
  - ECS Fargate
  - IAM
  - VPC/Subnet/Security Groups
  - Secrets Manager or SSM
- Docker & Git installed
- New Relic account + Ingest license key (EU region)
- If all is set and you have the lincese key from new-relic follow the steps to set up the project

### 1. Clone the repo

```bash
git clone https://github.com/MohamedAboMousallam/node-hello
cd node-hello
cd infra (if you want to provision terraform resources)
```

### 2. Create a file called terraform.tfvars with

```HCL
app_name             = "hello-node-app"
region               = "eu-north-1" #(Preferred)
container_image      = "docker.io/yourusername node-hello:latest"
container_port       = 3000
subnet_ids           = ["subnet-abc123", "subnet-def456"]
security_group_id    = "sg-xyz789"
execution_role_arn   = "arn:aws:iam::<account-id>:role/hello-node-app-ecs-task-execution-role"
role_attachment_id   = "aws_iam_role_policy_attachment.ecs_task_exec_attach"
newrelic_license_key = "your_new_relic_ingest_key"
```

### make sure to replace all the placeholders with your correct values

### Deploy your infrastructure resources

```bash
terraform init
terraform apply
```

## Infrastructure Design (via Terraform)

Infrastructure is modularized under the `infra/` directory and follows Terraform best practices:

### Modules

- **vpc/**: Creates VPC, public subnets, Internet Gateway, Route Table, and Security Group.
- **iam/**: Sets up ECS task execution IAM role and attaches necessary permissions.
- **ecs/**: Manages ECS cluster, task definitions (main app + FireLens sidecar), and ECS service.

### Root Module (infra/main.tf)

- Composes the above modules and passes necessary variables.
- Handles variable definitions, environment separation, and remote logging setup.

### Remote Logging (FireLens + New Relic)

- The `log_router` sidecar container uses Fluent Bit with FireLens to forward logs.
- Logs are shipped directly to New Relic's EU endpoint using:

  ```json
  {
    "Name": "newrelic",
    "apiKey": "<from SSM>",
    "endpoint": "https://log-api.eu.newrelic.com/log/v1"
  }
  ```

---

## CI/CD Pipeline

GitHub Actions is used for Continuous Integration:

### Workflow Includes:

- **Prettier** and **ESLint** checks on each push and pull request.
- Build and push Docker image to Docker Hub.

---

## Deployment Flow

1. Code is pushed to GitHub.
2. GitHub Actions runs linting, formatting, and Docker build.
3. Docker image is published to Docker Hub.

---

## Security Considerations

- New Relic secret is stored securely in **SSM Parameter Store**, not in plaintext or version control.

---

## ✅ Completed Features

- Containerized Node.js app
- CI/CD pipeline with GitHub Actions
- Prettier and ESLint integrated in CI pipeline
- ECS Fargate deployment via Terraform
- New Relic integration using FireLens
- Secure secret management via SSM
- Logs forwarded to New Relic and visible in UI
