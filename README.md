# github-templates
Reusable "called" workflows that can be invoked from other workflows "callers". 

Currently, in GitHub Actions, you cannot directly use a reusable workflow as a step within a job in your main workflow. Reusable workflows are designed to be used at the job level, meaning you can only call them as entire jobs, not as individual steps within a job.

For example: 

```
name: "Example Workflow"

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  setup-gcloud:
    uses: abridgeai/github-templates/.github/workflows/setup-gcloud.yml@main
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    with:
      GCP_SA_EMAIL: ${{ vars.GCP_SA_EMAIL_OPERATIONS }}
      GCP_WORKLOAD_IDENTITY_PROVIDER: ${{ vars.GCP_WORKLOAD_IDENTITY_PROVIDER_OPERATIONS }}

  list-buckets:
    name: "List buckets"
    needs: setup-gcloud
    runs-on: ubuntu-latest
    steps:
      - name: List GCS Buckets
        run: |
          gcloud storage buckets list --project=${{ vars.GCP_PROJECT_ID }}

```

## Workload Identity Permissions 

Every github repo has it's own service account to use with workload identity. More information on how this works is available to read [here](https://github.com/google-github-actions/auth?tab=readme-ov-file#workload-identity-federation-through-a-service-account). 

Make sure the shared-infra Terraform for the environment you're wanting to deploy to from Github actions has the proper permissioning set up. For example, [here's](https://github.com/abridgeai/infrastructure/blob/main/tf-modules/terraform-abridge-operations-shared-infra/github-auth.tf) how the `jit-access` Github repo is set up to deploy to the operations GCP project using the `gh-jit-access` service account. 

## App Engine Deploy 

[Example](examples/sample-app-engine-deploy.yml) of invoking the `app-engine-deploy.yml` shared workflow

## Cloud Function Deploy 

[Example](examples/sample-cloud-function-deploy.yml)  of invoking the `cloud-function-deploy.yml` shared workflow

## Setup gcloud CLI 

[Example](examples/sample-setup-gcloud.yml)