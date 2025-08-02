## Assumptions

Before implementing the solution, I made a few assumptions to guide my submission in alignment with best practices:

1. **Code Formatting & Quality**

   * A `.prettierrc` file was found in the repository, suggesting that consistent formatting was expected.
   * I added both **Prettier** and **ESLint** to ensure automatic code formatting and linting via GitHub Actions.

2. **Log Aggregation with New Relic**
   * Two approaches were considered (from the new-relic docs):
     * **S3 Bucket + Lambda Forwarder**
     * **Sidecar Container (FireLens)**
   * I selected the **FireLens Sidecar** approach for this assignment due to its real-time log shipping and direct integration with ECS.
3. **Secret Management**

   * Instead of hardcoding secrets (e.g., New Relic API key), I used **AWS SSM Parameter Store**.
   * The license key was securely fetched by the ECS task at runtime using `secretOptions` but it was commented because later I changed from JSON fomat to plain text (suggested by new-relic docs).

---

## Infrastructure Design (via Terraform)

Infrastructure is modularized under the `infra/` directory and follows Terraform best practices:

### Modules

* **vpc/**: Creates VPC, public subnets, Internet Gateway, Route Table, and Security Group.
* **iam/**: Sets up ECS task execution IAM role and attaches necessary permissions.
* **ecs/**: Manages ECS cluster, task definitions (main app + FireLens sidecar), and ECS service.

### Root Module (infra/main.tf)

* Composes the above modules and passes necessary variables.
* Handles variable definitions, environment separation, and remote logging setup.

### Remote Logging (FireLens + New Relic)

* The `log_router` sidecar container uses Fluent Bit with FireLens to forward logs.
* Logs are shipped directly to New Relic's EU endpoint using:

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

* **Prettier** and **ESLint** checks on each push and pull request.
* Build and push Docker image to Docker Hub.

---

## Deployment Flow

1. Code is pushed to GitHub.
2. GitHub Actions runs linting, formatting, and Docker build.
3. Docker image is published to Docker Hub.
---

## Security Considerations
* New Relic secret is stored securely in **SSM Parameter Store**, not in plaintext or version control.
---

## ✅ Completed Features

* Containerized Node.js app
* CI/CD pipeline with GitHub Actions
* Prettier and ESLint integrated in CI pipeline
* ECS Fargate deployment via Terraform
* New Relic integration using FireLens
* Secure secret management via SSM
* Logs forwarded to New Relic and visible in UI
