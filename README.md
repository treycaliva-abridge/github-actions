# github-templates
Reusable "called" workflows that can be invoked from other workflows "callers". 

Currently, in GitHub Actions, you cannot directly use a reusable workflow as a step within a job in your main workflow. Reusable workflows are designed to be used at the job level, meaning you can only call them as entire jobs, not as individual steps within a job.

## Workload Identity Permissions 

Every github repo has it's own service account to use with workload identity. More information on how this works is available to read [here](https://github.com/google-github-actions/auth?tab=readme-ov-file#workload-identity-federation-through-a-service-account). 

Make sure the shared-infra Terraform for the environment you're wanting to deploy to from Github actions has the proper permissioning set up. For example, [here's](https://github.com/abridgeai/infrastructure/blob/main/tf-modules/terraform-abridge-operations-shared-infra/github-auth.tf) how the `jit-access` Github repo is set up to deploy to the operations GCP project using the `gh-jit-access` service account. 

## App Engine Deploy 

[Example](examples/sample-app-engine-deploy.yml) of invoking the `app-engine-deploy.yml` shared workflow

## Cloud Function Deploy 

[Example](examples/sample-cloud-function-deploy.yml)  of invoking the `cloud-function-deploy.yml` shared workflow
