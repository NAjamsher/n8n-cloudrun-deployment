# n8n Self-Hosting on Google Cloud Run

A complete guide to deploying n8n workflow automation platform on Google Cloud Run with PostgreSQL persistence. This setup gives you full control over your automation workflows at minimal cost.

## What is n8n?

n8n is a workflow automation platform that lets you connect different apps and services together visually — without writing much code. Think of it like building automation pipelines: a form submission triggers an email, saves data to a spreadsheet, and notifies your team — all automatically.

## What We Built

| Component | Technology |
|---|---|
| Application | n8n (official Docker image) |
| Hosting | Google Cloud Run (serverless) |
| Database | Cloud SQL — PostgreSQL |
| Secrets Management | Google Secret Manager |
| Deployment Method | Terraform (automated) |
| Region | Configured via .env |

## Why Self-Host on Cloud Run?

- ✅ No monthly subscription fees
- ✅ Pay only when n8n is actually running
- ✅ Full control over your data and workflows
- ✅ Scales automatically
- ✅ Free to start with Google Cloud free credits

---

## Prerequisites

Before you begin make sure you have:

- A Google Cloud account with billing enabled
- [gcloud CLI](https://cloud.google.com/sdk/docs/install) installed and configured
- [Terraform](https://developer.hashicorp.com/terraform/install) installed
- [Git](https://git-scm.com/) installed
- Basic familiarity with the command line

---

## Environment Variables Setup

This project uses a `.env` file to manage all sensitive configuration. We use `python-dotenv` to load these variables securely.

### Step 1 — Install python-dotenv

```bash
pip install python-dotenv
```

### Step 2 — Create your .env file

Create a file named `.env` in the root of your project directory:

```bash
touch .env
```

Add the following variables to your `.env` file:

```env
# Google Cloud Configuration
GCP_PROJECT_ID=your-google-cloud-project-id
GCP_REGION=your-preferred-region

# Cloud Run Configuration
CLOUD_RUN_SERVICE_NAME=your-cloud-run-service-name
CLOUD_RUN_URL=your-cloud-run-url

# Database Configuration
DB_INSTANCE_NAME=your-db-instance-name
DB_NAME=your-database-name
DB_USER=your-database-user
DB_PASSWORD=your-secure-database-password

# n8n Configuration
N8N_ENCRYPTION_KEY=your-random-encryption-key
N8N_BASIC_AUTH_USER=your-n8n-username
N8N_BASIC_AUTH_PASSWORD=your-n8n-password
```

### Step 3 — Load environment variables in Python

```python
from dotenv import load_dotenv
import os

load_dotenv()

project_id = os.getenv("GCP_PROJECT_ID")
region = os.getenv("GCP_REGION")
cloud_run_url = os.getenv("CLOUD_RUN_URL")
```

### ⚠️ IMPORTANT — Never commit your .env file

Add `.env` to your `.gitignore` file immediately:

```bash
echo ".env" >> .gitignore
```

---

## Deployment Steps

### Step 1 — Set Up Google Cloud Project

Open Google Cloud Shell or your terminal with gcloud configured:

```bash
# Authenticate with Google Cloud
gcloud auth login

# Set your project
gcloud config set project $GCP_PROJECT_ID

# Enable all required APIs
gcloud services enable artifactregistry.googleapis.com
gcloud services enable run.googleapis.com
gcloud services enable sqladmin.googleapis.com
gcloud services enable secretmanager.googleapis.com
```

### Step 2 — Clone the Terraform Configuration

Clone the deployment repository and navigate to the terraform directory:

```bash
git clone <repository-url>
cd <repository-name>/terraform
```

### Step 3 — Configure Terraform Variables

Create a `terraform.tfvars` file with your project details:

```hcl
gcp_project_id = "your-project-id"
gcp_region     = "your-preferred-region"
```

> **Note:** Never commit `terraform.tfvars` to Git if it contains sensitive values. Add it to `.gitignore`.

### Step 4 — Initialize Terraform

```bash
terraform init
```

This downloads all required providers and modules.

### Step 5 — Review the Plan

```bash
terraform plan -var-file=terraform.tfvars
```

Review the output carefully — it shows every resource that will be created before anything is actually deployed.

### Step 6 — Deploy

```bash
terraform apply -var-file=terraform.tfvars
```

Type `yes` when prompted to confirm. Terraform will now:
- Create a Cloud SQL PostgreSQL instance
- Set up Secret Manager secrets
- Create a service account with correct permissions
- Deploy n8n to Cloud Run

This takes approximately **10–15 minutes** to complete.

### Step 7 — Access Your n8n Instance

Once deployment is complete, Terraform outputs your Cloud Run URL. Access n8n at:

```
https://<your-cloud-run-url>/home/workflows
```

> The base URL without `/home/workflows` will return "Cannot GET /" — always use the full path.

---

## Project Structure

```
.
├── terraform/
│   ├── main.tf              # Main Terraform configuration
│   ├── variables.tf         # Variable definitions
│   ├── outputs.tf           # Output values
│   └── terraform.tfvars     # Your values (DO NOT commit)
├── .env                     # Environment variables (DO NOT commit)
├── .env.example             # Example env file (safe to commit)
├── .gitignore               # Git ignore rules
└── README.md                # This file
```

---

## .env.example

Create a `.env.example` file that is safe to commit — it shows the structure without real values:

```env
# Google Cloud Configuration
GCP_PROJECT_ID=
GCP_REGION=

# Cloud Run
CLOUD_RUN_SERVICE_NAME=
CLOUD_RUN_URL=

# Database
DB_INSTANCE_NAME=
DB_NAME=
DB_USER=
DB_PASSWORD=

# n8n
N8N_ENCRYPTION_KEY=
N8N_BASIC_AUTH_USER=
N8N_BASIC_AUTH_PASSWORD=
```

---

## .gitignore

Make sure your `.gitignore` contains at minimum:

```gitignore
# Environment variables — never commit
.env

# Terraform sensitive files
*.tfvars
terraform.tfstate
terraform.tfstate.backup
.terraform/

# Credentials and keys
*.json
*.pem
*.key
```

---

## Cost Estimate

| Service | Cost |
|---|---|
| Cloud Run | Free when idle, ~$0 for low usage |
| Cloud SQL (db-f1-micro) | ~$7–10/month |
| Secret Manager | ~$0 for small usage |
| **Total** | **~$0 with free credits** |

Google Cloud gives new accounts **$300 in free credits** valid for 90 days — more than enough to run this setup for free.

---

## Troubleshooting

**"Cannot GET /" error**
Always access n8n via the full path: `/home/workflows`

**Container fails to start**
Cloud Run needs the container to respond on the correct port. n8n uses port `5678`. Ensure this is set correctly in your Terraform config.

**Database connection errors**
The n8n container needs a 5-second startup delay to allow the database connection to initialize. This is handled automatically in the Terraform configuration.

**Cold start delay**
Cloud Run scales to zero when idle. The first request after inactivity may take 10–20 seconds to respond. This is normal behaviour.

---

## What's Next

- [ ] Set up a custom domain
- [ ] Enable Google OAuth for connecting to Google Sheets, Drive etc.
- [ ] Create your first automation workflow
- [ ] Explore n8n templates at [n8n.io/workflows](https://n8n.io/workflows)

---

## References

- [n8n Official Documentation](https://docs.n8n.io)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Terraform Google Provider](https://registry.terraform.io/providers/hashicorp/google/latest/docs)
- [Original Deployment Guide](https://community.n8n.io/t/complete-guide-self-hosting-n8n-on-google-cloud-run-with-postgresql-serverless-cost-effective/195995)
