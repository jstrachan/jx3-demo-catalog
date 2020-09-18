# Google Terraform Quickstart template

Use this template to easily create a new Git Repository for managing Jenkins X cloud infrastructure needs.

We recommend using Terraform to manange the infrastructure needed to run Jenkins X.  There can be a number of cloud resources which need to be created such as:

- Kubernetes cluster
- Storage buckets for long term storage of logs
- IAM Bindings to manage permissions for applications using cloud resources

Jenkins X likes to use GitOps to manage the lifecycle of both infrastructure and cluster resources.  This requires two Git Repositories to achive this:
- the first, infrastructure resources will be managed by Terraform and will keep resourecs in sync.
- the second, the Kubernetes specific cluster resources will be managed by Jenkins X and keep resources in sync.

# Prerequisites

- Create a git bot user (different from your own personal user) and generate an access token, this will be used by Jenkins X to interact with git repositories
  e.g. https://github.com/settings/tokens/new?scopes=repo,read:user,read:org,user:email,write:repo_hook,delete_repo

  __This bot user needs to have write permission to write to any git repository used by Jenkins X.  This can be done by adding teh bot user to the git organisation level or individual repositories as a collaborator__

- Install `terraform` CLI - [see here](https://learn.hashicorp.com/tutorials/terraform/install-cli#install-terraform)
- Install `jx` CLI - [see here](https://github.com/jenkins-x/jx-cli/releases)

# Getting started

1. Create and clone your Infrastructure git repo from this GitHub Template https://github.com/jx3-gitops-repositories/jx3-terraform-gke/generate


2. Chose your desired secrets store, either Google Secret Manager or Vault and create your cluster git repository
    - __Google Secret Manager__: https://github.com/jx3-gitops-repositories/jx3-gke-gsm/generate

    - __Vault__: https://github.com/jx3-gitops-repositories/jx3-gke-terraform-vault/generate

Commit required terraform values from below to your `values.auto.tfvars`, e.g.

```sh
echo jx_git_url = "https://github.com/$git_owner_from_cluster_template_above/$git_repo_from_cluster_template_above" >> values.auto.tfvars
echo gcp_project = "my-cool-project" >> values.auto.tfvars
```
If using Google Secret Manager (not Vault) cluster template from above enable it for Terraform using:
```sh
echo gsm = true >> values.auto.tfvars
```

Now, initialise, plan and apply Terraform:

```sh
terraform init
```

```sh
terraform plan
```

```sh
terraform apply -var jx_bot_username=foo-bot -var jx_bot_token=abc123
```

Now follow the Terraform output next steps to track the creation of infrastructure and Jenkins X installation.

Once finished you can now move into the Jenkins X Developer namespace

```sh
jx ns jx
```

and create or import your applications

```sh
jx project
```

## Terraform Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| cluster\_location | The location (region or zone) in which the cluster master will be created. If you specify a zone (such as us-central1-a), the cluster will be a zonal cluster with a single cluster master. If you specify a region (such as us-west1), the cluster will be a regional cluster with multiple masters spread across zones in the region | `string` | `"us-central1-a"` | no |
| cluster\_name | Name of the Kubernetes cluster to create | `string` | `""` | no |
| gcp\_project | The name of the GCP project to use | `string` | n/a | yes |
| gsm | Enables Google Secrets Manager, not available with JX2 | `bool` | `false` | no |
| jx\_bot\_token | Bot token used to interact with the Jenkins X cluster git repository | `string` | n/a | yes |
| jx\_bot\_username | Bot username used to interact with the Jenkins X cluster git repository | `string` | n/a | yes |
| jx\_git\_url | URL for the Jenins X cluster git repository | `string` | n/a | yes |
| lets\_encrypt\_production | Flag to determine wether or not to use the Let's Encrypt production server. | `bool` | `true` | no |
| max\_node\_count | Maximum number of cluster nodes | `number` | `5` | no |
| min\_node\_count | Minimum number of cluster nodes | `number` | `3` | no |
| node\_disk\_size | Node disk size in GB | `string` | `"100"` | no |
| node\_disk\_type | Node disk type, either pd-standard or pd-ssd | `string` | `"pd-standard"` | no |
| node\_machine\_type | Node type for the Kubernetes cluster | `string` | `"n1-standard-2"` | no |
| parent\_domain | The parent domain to be allocated to the cluster | `string` | `""` | no |
| resource\_labels | Set of labels to be applied to the cluster | `map(string)` | `{}` | no |
| tls\_email | Email used by Let's Encrypt. Required for TLS when parent\_domain is specified | `string` | `""` | no |

# Cleanup

To remove any cloud resources created here run:
```sh
terraform destroy
```

# Contributing

When adding new variables please regenerate the markdown table 
```sh
terraform-docs markdown table .
```
and replace the Inputs section above

## Formatting

When developing please remember to format codebase before raising a pull request
```sh
terraform fmt -check -diff -recursive
```