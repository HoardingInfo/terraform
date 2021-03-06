---
layout: "enterprise"
page_title: "Runs: Variables and Configuration"
sidebar_current: "docs-enterprise-runs-variables"
description: |-
  How to configure runs and their variables.
---

# Terraform Variables and Configuration

There are two ways to configure Terraform runs – with
Terraform variables or environment variables.

## Terraform Variables

Terraform variables are first-class configuration in Terraform. They
define the parameterization of Terraform configurations and are important
for sharing and removal of sensitive secrets from version control.

Variables are sent with the `terraform push` command. Any variables in your local
`.tfvars` files are securely uploaded. Once variables are uploaded, Terraform will prefer the stored variables over any changes you
make locally. Please refer to the
[Terraform push documentation](https://www.terraform.io/docs/commands/push.html)
for more information.

You can also add, edit, and delete variables. To update
Terraform variables, visit the "variables" page on your
environment.

The maximum size for the value of Terraform variables is `256kb`.

For detailed information about Terraform variables, please read the
[Terraform variables](https://terraform.io/docs/configuration/variables.html)
section of the Terraform documentation.

## Environment Variables

Environment variables are injected into the virtual environment that Terraform
executes in during the `plan` and `apply` phases.

You can add, edit, and delete environment variables from the "variables" page
on your environment.

Additionally, the following environment variables are automatically injected by
Terraform Enterprise. All injected environment variables will be prefixed with `ATLAS_`

- `ATLAS_TOKEN` - This is a unique, per-run token that expires at the end of
  run execution (e.g. `"abcd.atlasv1.ghjkl..."`).
- `ATLAS_RUN_ID` - This is a unique identifier for this run (e.g. `"33"`).
- `ATLAS_CONFIGURATION_NAME` - This is the name of the configuration used in
  this run. Unless you have configured it differently, this will also be the
  name of the environment (e.g `"production"`).
- `ATLAS_CONFIGURATION_SLUG` - This is the full slug of the configuration used
  in this run. Unless you have configured it differently, this will also be the
  name of the environment (e.g. `"company/production"`).
- `ATLAS_CONFIGURATION_VERSION` - This is the unique, auto-incrementing version
  for the Terraform configuration (e.g. `"34"`).
- `ATLAS_CONFIGURATION_VERSION_GITHUB_BRANCH` - This is the name of the branch
  that the associated Terraform configuration version was ingressed from
  (e.g. `master`).
- `ATLAS_CONFIGURATION_VERSION_GITHUB_COMMIT_SHA` - This is the full commit hash
  of the commit that the associated Terraform configuration version was
  ingressed from (e.g. `"abcd1234..."`).
- `ATLAS_CONFIGURATION_VERSION_GITHUB_TAG` - This is the name of the tag
  that the associated Terraform configuration version was ingressed from
  (e.g. `"v0.1.0"`).

For any of the `GITHUB_` attributes, the value of the environment variable will
be the empty string (`""`) if the resource is not connected to GitHub or if the
resource was created outside of GitHub (like using `terraform push`).

## Managing Secret Multi-Line Files

Terraform Enterprise has the ability to store multi-line files as variables. The recommended way to manage your secret/sensitive multi-line files (private key, SSL cert, SSL private key, CA, etc.) is to add them as [Terraform Variables](#terraform-variables) or [Environment Variables](#environment-variables).

Just like secret strings, it is recommended that you never check in these multi-line secret files to version control by following the below steps.

Set the [variables](https://www.terraform.io/docs/configuration/variables.html) in your Terraform template that resources utilizing the secret file will reference:

    variable "private_key" {}

    resource "aws_instance" "example" {
      ...

      provisioner "remote-exec" {
        connection {
          host         = "${self.private_ip}"
          private_key  = "${var.private_key}"
        }

        ...
      }
    }

`terraform push` any "Terraform Variables":

    $ terraform push -name $ATLAS_USERNAME/example -var "private_key=$MY_PRIVATE_KEY"

`terraform push` any "Environment Variables":

    $ TF_VAR_private_key=$MY_PRIVATE_KEY terraform push -name $ATLAS_USERNAME/example

Alternatively, you can add or update variables manually by going to the "Variables" section of your Environment and pasting the contents of the file in as the value.

Now, any resource that consumes that variable will have access to the variable value, without having to check the file into version control. If you want to run Terraform locally, that file will still need to be passed in as a variable in the CLI. View the [Terraform Variable Documentation](https://www.terraform.io/docs/configuration/variables.html) for more info on how to accomplish this.

A few things to note...

The `.tfvars` file does not support multi-line files. You can still use `.tfvars` to define variables, however, you will not be able to actually set the variable in `.tfvars` with the multi-line file contents like you would a variable in a `.tf` file.

If you are running Terraform locally, you can pass in the variables at the command line:

    $ terraform apply -var "private_key=$MY_PRIVATE_KEY"
    $ TF_VAR_private_key=$MY_PRIVATE_KEY terraform apply

You can update variables locally by using the `-overwrite` flag with your `terraform push` command:

    $ terraform push -name $ATLAS_USERNAME/example -var "private_key=$MY_PRIVATE_KEY" -overwrite=private_key
    $ TF_VAR_private_key=$MY_PRIVATE_KEY terraform push -name $ATLAS_USERNAME/example -overwrite=private_key

- - -

## Notes on Security

Terraform variables and environment variables are encrypted using
[Vault](https://vaultproject.io) and closely guarded and audited. If you have
questions or concerns about the safety of your configuration, please contact
our security team at [security@hashicorp.com](mailto:security@hashicorp.com).
