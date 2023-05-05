# Example multi-app terraform structure

This is an example of basic terraform structure fitting for most infrastructure. Below you may find the description of and some basic assumptions about the workflow.

An application ([\[1\]][1], [\[2\]][2], [\[3\]][3]) is a word used throughout this example, which could translate to a single terraform state and can communicate with other applications through a remote state connection. It is a single piece of infrastructure, sometimes refered to as a stack, maybe a workspace or a template. 

The structure tries to avoid touching terraform workflows, in order to allow flexibility for a seasoned engineer to:
- promote changes from one environment to another and
- separate infrastructure for various environments

For easier explanation, github actions are assumed as the engine behind the terraform workflow. But any other can be used, and it might better suit your needs.
The following things are not covered in this example:
- dependency management between applications
- handling secrets
- linting / testing

## Requirements

- AWS credentials / role to assume: the workflow currently supports only AWS backed Terraform
- Provider lock files should be set for all architectures: `terraform providers lock -platform=windows_amd64 -platform=darwin_amd64 -platform=linux_amd64`

### Terraform structure
Following is the example structure proposed in this repository:

```
terraform-basics/
├─ dev/
│  ├─ backend.s3.tfvars
│  ├─ env.tfvars
│  ├─ another-app/
│  │  ├─ main.tf
|  |  ├─ pre-terraform-plan.sh
│  │  ├─ README.md
│  ├─ great-app/
│  │  ├─ main.tf
│  │  ├─ README.md
├─ prod/
│  ├─ backend.s3.tfvars
│  ├─ env.tfvars
│  ├─ another-app/
│  │  ├─ README.md
│  │  ├─ override.tf
│  ├─ basic-infra/
│  │  ├─ main.tf
│  │  ├─ README.md
├─ .gitignore
├─ applications.yaml
├─ README.md
```

Root folder is divided into environments (dev, stage, prod). Each environment contains a number of apps, which are separate terraform entities. They:
- **MUST** have separate backend configuration (especially the terraform state).
- **MUST** contain a readme file
- **MAY** source from other apps using remote state resources. Since this creates dependencies between apps we **MUST** be carefull not to introduce loops.
- **MAY** use apps from other environments as modules with applied overrides. See [next chapters for more explanation](#using-other-apps-as-modules).
- **MAY** symlink to files in other apps.
- **MUST NOT** symlink to other apps, because you will get into troubles with relative paths.

There are two (could be more) relevant files in root folders, which shall be loaded by the terraform workflow:
- `backend.*.tfvars` contains all but the `path` parameter for the terraform backend configuration
- `env.tfvars` contains environment specific variables

https://github.com/terraform-linters/tflint-ruleset-opa

### Using other apps as modules
Terraform will issue a warning in this case, since an app is not a proper module (it contains a backend definition, for example). This is furt

### A word about dependencies
Root folder includes the `applications.yaml` file, which contains dependencies between various apps. Here we can configure how the CI will process them. See [examples](https://github.com/dorny/paths-filter) or tests next to them for hints how to use the file!




## References

- \[1]: [A similar notion to "applications" from terraform cloud][1]
- \[2]: [A similar notion to "applications" from env0][2]
- \[3]: [A similar notion to "applications" from terraspace][3]
- \[4]: [Terraform best practices][4]

[1]: https://developer.hashicorp.com/terraform/cloud-docs/recommended-practices/part1#one-workspace-per-environment-per-terraform-configuration
[2]: https://docs.env0.com/docs/templates
[3]: https://terraspace.cloud/docs/intro/deploy-all/
[4]: https://cloud.google.com/docs/terraform/best-practices-for-terraform
