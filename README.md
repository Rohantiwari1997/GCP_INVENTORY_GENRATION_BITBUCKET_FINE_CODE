# GCP Inventory Generator

This tool scans one or more Google Cloud Platform (GCP) projects to generate a comprehensive inventory of resources. The results are exported to an Excel file and optionally uploaded to a Google Cloud Storage (GCS) bucket.

## Features
- **Comprehensive Scanning**: Captures Compute Instances, GKE Clusters, Cloud Functions, Cloud SQL Instances, and Storage Buckets.
- **Cloud Asset Inventory**: Option to use the Cloud Asset API for a deep, cross-region search of all resources.
- **Excel Export**: Generates a formatted `.xlsx` file with separate sheets for each resource type.
- **Automated Pipeline**: Includes a Bitbucket Pipeline configuration for automated execution.
- **Security**: Supports Service Account authentication and secure Base64-encoded credentials for CI/CD.

## Files Structure
- `inventory.py`: Core logic for fetching resources and generating the Excel report.
- `install_and_run.sh`: Wrapper script that handles dependency installation and execution (ideal for CI/CD).
- `setup_service_account.sh`: Utility to create a Service Account, assign IAM roles, and generate a key file.
- `bitbucket-pipelines.yml`: CI/CD configuration file.
- `requirements.txt`: Python package dependencies.
- `.gitignore`: Ensures secrets and temporary files are not committed.

---

## Local Usage

### 1. Prerequisites
- Python 3.9+
- Google Cloud SDK (`gcloud`) installed and authenticated.
- A user or service account with `Viewer` and `Cloud Asset Viewer` permissions.

### 2. Installation
```bash
pip install -r requirements.txt
```

### 3. Running the Inventory
**Basic Run:**
```bash
python3 inventory.py --project <PROJECT_ID>
```

**With GCS Upload:**
```bash
python3 inventory.py --project <PROJECT_ID> --bucket <BUCKET_NAME>
```

**Using Cloud Asset Inventory (Recommended):**
```bash
python3 inventory.py --project <PROJECT_ID> --bucket <BUCKET_NAME> --use-asset
# Note: Requires the 'Cloud Asset API' to be enabled on the project.
```

---

## Bitbucket Pipeline Setup

This project is configured to run automatically on Bitbucket Pipelines. It uses a secure, optimized approach with the `google/cloud-sdk` Docker image.

### 1. Preparation
You need a Service Account Key to authenticate the pipeline. Use the provided utility script to create one if you haven't already:

```bash
./setup_service_account.sh <PROJECT_ID> [OPTIONAL_SA_NAME]
# This generates 'gcp-inventory-sa-key.json' in your current directory.
```

### 2. Repository Variables
Go to **Repository Settings > Pipelines > Repository variables** in Bitbucket and add the following:

| Variable Name | Description | Secured? |
| :--- | :--- | :--- |
| `GCP_PROJECT_ID` | Comma-separated list of GCP Project IDs to scan (e.g., `proj-a,proj-b`). | No |
| `GCS_BUCKET_NAME` | Name of the GCS bucket to upload the report to. | No |
| `GCP_SA_KEY_BASE64` | **Base64 encoded** content of your JSON key file. | **Yes** 🔒 |

### 3. How to Generate `GCP_SA_KEY_BASE64`
Do **not** commit the `.json` key file. Instead, encode it and store it as a variable:

**Mac/Linux:**
```bash
cat gcp-inventory-sa-key.json | base64 | tr -d '\n' | pbcopy
# Paste the clipboard content into the Bitbucket variable value.

or 

cat gcp-inventory-sa-key.json | base64 
```

---

## Technical Details

### Dependencies
The project uses `pandas` and `openpyxl` for Excel generation, and `google-cloud-asset` for API interaction. A `requirements.txt` is provided.

### Permissions
The Service Account used (either locally or in the pipeline) requires the following IAM roles:
- `roles/viewer` (or specifc read permissions)
- `roles/cloudasset.viewer`
- `roles/storage.objectAdmin` (for uploading reports)
- `roles/serviceusage.serviceUsageConsumer`

### CI/CD Optimization
The pipeline uses the official `google/cloud-sdk:slim` image to minimize build time. It handles `pip` caching and uses `--break-system-packages` to comply with modern Debian/Python environments in a CI context.
