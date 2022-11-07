# GCP Agentless POC (Same Account Model, not reusing SaaS credentials)

> In this document, the GCP Project and Prisma Cloud Account are named `example_project`.


## In GCP, in your Project:

* Enable Cloud Resource Manager API 
* Enable Compute Engine API
* Enable Identity and Access Management API
* Enable Deployment Manager API (to execute `gcloud deployment-manager` commands to apply templates)
* Temporarily add the `Roles > Role Administrator` and `IAM > Security Admin` Roles to the `Google APIs Service Agent` (aka `<project_number>@cloudservices.gserviceaccount.com`)
* Create an IAM Service Account and create and download a Service Account Key

For example, in CloudShell:

```
gcloud config set project example_project

gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable deploymentmanager.googleapis.com

gcloud projects add-iam-policy-binding \
    example_project \
    --member=serviceAccount:example_project_number@cloudservices.gserviceaccount.com \
    --role=roles/iam.roleAdmin

gcloud projects add-iam-policy-binding \
    example_project \
    --member=serviceAccount:example_project_number@cloudservices.gserviceaccount.com \
    --role=roles/iam.securityAdmin

gcloud iam service-accounts create \
    example_project \
    --display-name="Prisma Cloud Service Account for Agentless Scanning"

gcloud iam service-accounts keys create \
    example_project-service_account_key.json \
    --iam-account=example_project@example_project.iam.gserviceaccount.com
```

Download the Service Account Key (`example_project-service_account_key.json`)

> If the APIs are not enabled and the Roles are not added to the `Google APIs Service Agent`, the subsequent `gcloud deployment-manager deployments create` command will fail. If the `gcloud deployment-manager deployments create` command fails, enable the APIs and add the Roles and retry the command ... but replace the `create` with `update` if you receive a "deployment already exists" error.


## In Prisma Cloud, in Compute > Cloud Accounts, onboard the account:

* Click Add Account
* In the Service Account field, paste the Service Account Key (`example_project-service_account_key.json`) and leave the API Key field blank.
* Download the permission templates.
* Select Advanced Settings > Scanning Type > Same Account 


## In GCP, apply the permission templates:

* Upload all of the downloaded (and expanded) permission templates.

In CloudShell:

```
gcloud deployment-manager deployments create pc-agentless-hub-user-local --project example_project --template example_project_target_user_permissions.yaml.jinja
```


## In Prisma Cloud, in Compute > Cloud Accounts, test Agentless Scanning:

* Click Start Agentless Scan.
* Monitor the scan, looking for errors in Scanning Job Progress, or summarized in the list of Cloud Accounts.
* View the Agentless scan results in Compute > Monitor > Vulnerabilities > Hosts


## In GCP, in your Project, remove temporary Roles:

* In IAM, remove the `Roles > Role Administrator` and `IAM > Security Admin` Roles from the `Google APIs Service Agent` (aka `<project_number>@cloudservices.gserviceaccount.com`).

In CloudShell:

```
gcloud projects remove-iam-policy-binding \
    example_project \
    --member=serviceAccount:example_project_number@cloudservices.gserviceaccount.com \
    --role=roles/iam.roleAdmin
    
gcloud projects remove-iam-policy-binding \
    example_project \
    --member=serviceAccount:example_project_number@cloudservices.gserviceaccount.com \
    --role=roles/iam.securityAdmin
```

> Those roles were only needed to execute `gcloud deployment-manager` commands to apply the permission templates.


## Notes

* To speed up testing, create VMs in just one Region, and configure Prisma Cloud to scan that Region via `Custom Regions`.
* You cannot use a `default` subnet (or specify a custom subnet) provided by a Shared VPC (https://prismacloud.ideas.aha.io/ideas/PANW-I-4147)
