# github-templates
Reusable "called" workflows that can be invoked from other workflows "callers". 

Currently, in GitHub Actions, you cannot directly use a reusable workflow as a step within a job in your main workflow. Reusable workflows are designed to be used at the job level, meaning you can only call them as entire jobs, not as individual steps within a job.

## Creating a New Repo 
You should have access to create new repos within the `abridgeai` organization in Github. Reach out to Platform team if you don't have access to do so. 

## Migrating a Repo from Bitbucket to Github

1. Reach out to Platform team to have them clone the repo into Github
2. Provide list of users to platform team on who should have access to the repo.
3. Make sure [workload identity permissions](#workload-identity-permissions) are set up correctly (you can make your own Terraform PR or contact Platform team so they can add them). You'll probably need to reference the roles attached to the GCP Service Account currently being used by your pipeline in Bitbucket.
4. Migrate `bitbucket-pipeline.yaml` to Github Action Workflows. See [Github Workflows](#github-workflows). Please consult with Platform team if you run into any issues.
5. Deprecate Bitbucket repo
  - Move to `Graveyard` Bitbucket project
  - Add repo description calling it out as migrated. [Example](https://bitbucket.org/abridge-ai/heimdall/src/main/)

## Workload Identity Permissions 

Every github repo has it's own service account to use with workload identity. More information on how this works is available to read [here](https://github.com/google-github-actions/auth?tab=readme-ov-file#workload-identity-federation-through-a-service-account). As part of moving your pipeline, you'll need to make sure this new auth method can do what your SA was doing in Bitbucket before. If you need help identifying the needed roles, reach out to Platform team for help. 

Make sure the shared-infra Terraform for the environment you're wanting to deploy to from Github actions has the proper permissioning set up. You'll need to add your Github repo (must already exist) to the `repo_sa_mapping` variable in each GCP project your Github workflow(s) will interact with. For example: 

```
  repo_sa_mapping = {
    # Github repository name
    "my-repo-name" = {
      # service account name to create in project
      sa_name = "gh-my-repo-name"
      # roles granted to service account in project
      roles = [
        "roles/appengine.deployer",
        "roles/appengine.serviceAdmin",
        ...
      ]
    },
    "my-repo-name-2" = {
      # service account name to create in project
      sa_name = "gh-my-repo-name-2"
      # roles granted to service account in project
      roles = [
        "roles/appengine.deployer",
        "roles/appengine.serviceAdmin",
        ...
      ]
    },
  }

```
Here are links to the pertinent Terraform sections. 
Github Auth Terraform Setup By GCP Project: 
- [abridge-artifact-registry](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/artifact-registry/shared-github-auth/terragrunt.hcl#L19)
- [client-dev-e301d (development)](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/development/shared-github-auth/terragrunt.hcl#L19)
- [abridge-client-staging](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/staging/shared-github-auth/terragrunt.hcl#L19)
- [abridge-client-production](https://github.com/abridgeai/infrastructure/blob/4fd570ac41eb4022595227bf2bc179190e855308/tf-live/production/shared-github-auth/terragrunt.hcl#L19)

## Github Workflows 

### Getting Started With Workflows

Github has great [documentation](https://docs.github.com/en/actions/using-workflows/about-workflows) on setting up your workflows. Here are some examples currently used at Abridge that you can reference depending on your needs: 
- [Example App Engine Deploy](https://github.com/abridgeai/github-templates/blob/main/examples/sample-app-engine-deploy.yml)
- [Example Cloud Function Deploy](https://github.com/abridgeai/github-templates/blob/main/examples/sample-app-engine-deploy.yml)
- [Python Library Build and Publish to Artifact Registry](https://github.com/abridgeai/heimdall/blob/main/.github/workflows/deploy.yml)
- [App Engine Deploy to Single GCP Project](https://github.com/abridgeai/jit-access/blob/main/.github/workflows/deploy-prod.yml)
- [Multi-environment Cloud Function, Scheduler, & App Engine Deploy w/ Integration Tests](https://github.com/abridgeai/data-governance/tree/main/.github/workflows)

### Authentication to GCP w/ Workload Identity

Since we use workload identity to Authenticate to GCP from Github, there's no longer a need for Service Account Keys to be saved as repo variables. Instead, you can use the following as a step to auth to GCP: 

```
- id: "auth"
  uses: "google-github-actions/auth@v2"
  with:
    workload_identity_provider: ${{ vars.GCP_WORKLOAD_IDENTITY_PROVIDER_ARTIFACT_REGISTRY }}
    service_account: ${{ vars.GCP_SA_EMAIL_ARTIFACT_REGISTRY }}
```

The variables are automatically added to your repo after the auth has been set up in Terraform [above](#workload-identity-permissions). They will be in this format: 

```
GCP_WORKLOAD_IDENTITY_PROVIDER_<ENV_NAME>

```
Here are all current options: 
```
# Workload Identity
GCP_WORKLOAD_IDENTITY_PROVIDER_ARTIFACT_REGISTRY
GCP_WORKLOAD_IDENTITY_PROVIDER_DEVELOPMENT
GCP_WORKLOAD_IDENTITY_PROVIDER_STAGING
GCP_WORKLOAD_IDENTITY_PROVIDER_PRODUCTION

# SA Email
GCP_SA_EMAIL_ARTIFACT_REGISTRY
GCP_SA_EMAIL_DEVELOPMENT
GCP_SA_EMAIL_STAGING
GCP_SA_EMAIL_PRODUCTION
```

### Deploy Steps (CD)

It's highly recommended to leverage [Google's Github Actions](https://github.com/google-github-actions) where possible as they maintain these integrations regularly. Here are the main ones we recommend using: 

- `deploy-appengine` -- [https://github.com/google-github-actions/deploy-appengine](https://github.com/google-github-actions/deploy-cloud-functions)
- `deploy-cloudfunctions` -- [https://github.com/google-github-actions/deploy-cloud-functions](https://github.com/google-github-actions/deploy-cloud-functions)

### Shared Workflows

Shared workflows are mechanisms for reusing the same workflow from multiple repos. These can be referenced within the same repo (see the `data-governance` repo for an [example](https://github.com/abridgeai/data-governance/tree/main/.github/workflows) of this) or from a centralized repo in the org -- [`github-templates`](https://github.com/abridgeai/github-templates) (see `jit-access` repo for an [example](https://github.com/abridgeai/jit-access/blob/main/.github/workflows/deploy-prod.yml) of doing this). 

If you believe your workflow setup could be used across the org, feel free to add it to the `github-templates` repo in `.github/workflows`. 

Here are some basic examples:

- [Example](examples/sample-app-engine-deploy.yml) of invoking the `app-engine-deploy.yml` shared workflow

- [Example](examples/sample-cloud-function-deploy.yml)  of invoking the `cloud-function-deploy.yml` shared workflow
