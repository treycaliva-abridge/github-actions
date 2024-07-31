# github-templates
Reusable "called" workflows that can be invoked from other workflows "callers". 

Currently, in GitHub Actions, you cannot directly use a reusable workflow as a step within a job in your main workflow. Reusable workflows are designed to be used at the job level, meaning you can only call them as entire jobs, not as individual steps within a job.

## Creating a New Repo 
You should have access to create new repos within the `abridgeai` organization in Github. Reach out to Platform team if you don't have access to do so. 

## Migrating a Repo from Bitbucket to Github

1. Reach out to Platform team to have them clone the repo into Github
2. Provide list of users to platform team on who should have access to the repo.
3. Make sure [workload identity permissions](#workload-identity-permissions) are set up correctly (you can make your own Terraform PR or contact Platform team so they can add them). You'll probably need to reference the roles attached to the GCP Service Account currently being used by Bitbucket.
4. Migrate `bitbucket-pipeline.yaml` to Github Action Workflows. See [Github Workflows](#github-workflows). Consult with Platform team if you run into any issues. 

## Workload Identity Permissions 

Every github repo has it's own service account to use with workload identity. More information on how this works is available to read [here](https://github.com/google-github-actions/auth?tab=readme-ov-file#workload-identity-federation-through-a-service-account). 

Make sure the shared-infra Terraform for the environment you're wanting to deploy to from Github actions has the proper permissioning set up. 

Github Auth Terraform Setup Links By GCP Project: 
- [abridge-artifact-registry](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/artifact-registry/shared-github-auth/terragrunt.hcl#L19)
- [client-dev-e301d (development)](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/development/shared-github-auth/terragrunt.hcl#L19)
- [abridge-client-staging](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/staging/shared-github-auth/terragrunt.hcl#L19)
- [abridge-client-production](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/production/shared-github-auth/terragrunt.hcl#L19)

## Github Workflows 

### App Engine Deploy 

[Example](examples/sample-app-engine-deploy.yml) of invoking the `app-engine-deploy.yml` shared workflow

### Cloud Function Deploy 

[Example](examples/sample-cloud-function-deploy.yml)  of invoking the `cloud-function-deploy.yml` shared workflow
