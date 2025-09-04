# Terraform + GitHub Actions (using Secrets) - Automatic AWS deployment

> Based on my article: **Automating AWS Resource Deployment with GitHub Actions and Terraform**: https://www.playingaws.com/posts/automating-aws-resource-deployment-with-github-actions-and-terraform/

This repository demonstrates deploying **Terraform** to **AWS** via **GitHub Actions** using **Access Keys in GitHub Secrets** (legacy approach). It provisions a tiny **AWS Budget** (USD 0.1/mo) as a canary to validate the CI/CD flow.

> 🔐 **Security note**: The recommended method today is **OIDC** (no long‑lived secrets). Consider migrating — see the article above. [Coming soon] I will create a specific article for the migration.

## What’s inside

- `main.tf`: AWS provider, optional S3/DynamoDB **remote backend**, and an `aws_budgets_budget` resource.
- `.github/workflows/terraform-deploy.yml`: single job that runs `init`, `plan`, and `apply` on push to `main` using **secrets**.

---

## Requirements

- **AWS account** with permissions for S3 (state), DynamoDB (lock), and **AWS Budgets**.
- **GitHub repository** with Actions enabled.
- **Terraform** (≥ 1.5 recommended) if you want to test locally.

### Configure GitHub Secrets

Add these repository **Actions Secrets**:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

Workflow snippet:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: eu-west-1
```

---

## Repository structure

```code
.
├── main.tf
└── .github/
    └── workflows/
        └── terraform-deploy.yml
```

---

## Local test (optional)

```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_REGION=eu-west-1

terraform init
terraform plan
terraform apply -auto-approve
# terraform destroy -auto-approve
```

---

## Troubleshooting

- **AccessDenied on state** → ensure the user/role mapped to your keys has S3/DynamoDB permissions.  
- **Plan applies automatically** → this workflow applies on push; if you prefer approvals, split plan/apply or add environments.  
- **Want to modernize** → migrate to **OIDC** to remove static secrets.

---

## Costs

- **Budgets**: free.  
- **S3** (state) and **DynamoDB** (lock): minimal usage‑based cost.

## Cleanup

1) `terraform destroy -auto-approve`  
2) Delete S3 bucket (including versions) and DynamoDB table if not needed.

---

## References

- Full guide: https://www.playingaws.com/posts/automating-aws-resource-deployment-with-github-actions-and-terraform/

## Author

Created by **Alejandro Lázaro**. More on my blog: **https://www.playingaws.com**.
