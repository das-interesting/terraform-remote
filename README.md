# Terraform Remote State (Backend already deployed)

This repository **does not create the backend**. The AWS pieces you need are **already live**:

- S3 bucket for Terraform state
- DynamoDB table for state locking
- KMS key for encryption

We removed the bootstrap Infrastructure-as-Code to keep the repo small, but you can restore it from a safety tag if you ever need to make changes.

---

## If you need the original bootstrap code
Latest safety tag: `infra-bootstrap-20251017-211937`

```bash
git fetch --tags
git checkout -b restore-bootstrap infra-bootstrap-20251017-211937
# make any updates, then run:
# terraform init
# terraform plan
# terraform apply
```

---

## How to point another repo at this backend (step by step)
1) Copy the example backend config and use it as your local file:
   ```bash
   cp backend/dev.example.hcl backend/dev.hcl
   ```

2) Open `backend/dev.hcl` and fill in real values for:
   - `<STATE_BUCKET_NAME>` (the existing S3 bucket name)
   - `<STATE_KEY_PATH>` (e.g., `my-repo/dev/terraform.tfstate`)
   - `<AWS_REGION>` (e.g., `us-east-1`)
   - `<DDB_LOCK_TABLE_NAME>` (the existing DynamoDB lock table)
   - `<KMS_KEY_ARN>` (the existing KMS key ARN)

3) In your Terraform code, keep the backend block empty (Terraform fills it using your file):
   ```hcl
   terraform { backend "s3" {} }
   ```

4) Initialize Terraform with your backend settings:
   ```bash
   terraform init -reconfigure -backend-config=backend/dev.hcl
   ```

---

## What to commit (and not commit)
- **Commit:** `backend/dev.example.hcl` and `.terraform.lock.hcl` (locks provider versions).
- **Do not commit:** `backend/dev.hcl` (it will contain real AWS values).
- Security: the backend enforces KMS encryption and DynamoDB locking by default.
