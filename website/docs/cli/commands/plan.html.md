---
layout: "docs"
page_title: "Command: plan"
sidebar_current: "docs-commands-plan"
description: |-
  The `terraform plan` command is used to create an execution plan. Terraform performs a refresh, unless explicitly disabled, and then determines what actions are necessary to achieve the desired state specified in the configuration files. The plan can be saved using `-out`, and then provided to `terraform apply` to ensure only the pre-planned actions are executed.
---

# Command: plan

> **Hands-on:** Try the [Terraform: Get Started](https://learn.hashicorp.com/collections/terraform/aws-get-started?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS) collection on HashiCorp Learn.

The `terraform plan` command is used to create an execution plan. Terraform
performs a refresh, unless explicitly disabled, and then determines what
actions are necessary to achieve the desired state specified in the
configuration files.

This command is a convenient way to check whether the execution plan for a
set of changes matches your expectations without making any changes to
real resources or to the state. For example, `terraform plan` might be run
before committing a change to version control, to create confidence that it
will behave as expected.

The optional `-out` argument can be used to save the generated plan to a file
for later execution with `terraform apply`, which can be useful when
[running Terraform in automation](https://learn.hashicorp.com/tutorials/terraform/automate-terraform?in=terraform/automation&utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS).

If Terraform detects no changes to resource or to root module output values,
`terraform plan` will indicate that no changes are required.

## Usage

Usage: `terraform plan [options]`

The `plan` subcommand looks in the current working directory for the root module
configuration.

The available options are:

* `-compact-warnings` - If Terraform produces any warnings that are not
  accompanied by errors, show them in a more compact form that includes only
  the summary messages.

* `-destroy` - If set, generates a plan to destroy all the known resources.

* `-detailed-exitcode` - Return a detailed exit code when the command exits.
  When provided, this argument changes the exit codes and their meanings to
  provide more granular information about what the resulting plan contains:
  * 0 = Succeeded with empty diff (no changes)
  * 1 = Error
  * 2 = Succeeded with non-empty diff (changes present)

* `-input=true` - Ask for input for variables if not directly set.

* `-lock=true` - Lock the state file when locking is supported.

* `-lock-timeout=0s` - Duration to retry a state lock.

* `-no-color` - Disables output with coloring.

* `-out=path` - The path to save the generated execution plan. This plan
  can then be used with `terraform apply` to be certain that only the
  changes shown in this plan are applied. Read the warning on saved
  plans below.

* `-parallelism=n` - Limit the number of concurrent operation as Terraform
  [walks the graph](/docs/internals/graph.html#walking-the-graph). Defaults
  to 10.

* `-refresh=true` - Update the state prior to checking for differences.

* `-target=resource` - A [Resource
  Address](/docs/cli/state/resource-addressing.html) to target. This flag can
  be used multiple times. See below for more information.

* `-var 'foo=bar'` - Set a variable in the Terraform configuration. This flag
  can be set multiple times. Variable values are interpreted as
  [literal expressions](/docs/language/expressions/types.html) in the
  Terraform language, so list and map values can be specified via this flag.

* `-var-file=foo` - Set variables in the Terraform configuration from
  a [variable file](/docs/language/values/variables.html#variable-definitions-tfvars-files). If
  a `terraform.tfvars` or any `.auto.tfvars` files are present in the current
  directory, they will be automatically loaded. `terraform.tfvars` is loaded
  first and the `.auto.tfvars` files after in alphabetical order. Any files
  specified by `-var-file` override any values set automatically from files in
  the working directory. This flag can be used multiple times.

For configurations using
[the `local` backend](/docs/language/settings/backends/local.html) only,
`terraform plan` accepts the legacy command line option
[`-state`](/docs/language/settings/backends/local.html#command-line-arguments).

## Resource Targeting

The `-target` option can be used to focus Terraform's attention on only a
subset of resources.
[Resource Address](/docs/cli/state/resource-addressing.html) syntax is used
to specify the constraint. The resource address is interpreted as follows:

* If the given address has a _resource spec_, only the specified resource
  is targeted. If the named resource uses `count` and no explicit index
  is specified in the address (i.e. aws_instance.example[3]), all of the instances sharing the given
  resource name are targeted.

* If the given address _does not_ have a resource spec, and instead just
  specifies a module path, the target applies to all resources in the
  specified module _and_ all of the descendent modules of the specified
  module.

This targeting capability is provided for exceptional circumstances, such
as recovering from mistakes or working around Terraform limitations. It
is *not recommended* to use `-target` for routine operations, since this can
lead to undetected configuration drift and confusion about how the true state
of resources relates to configuration.

Instead of using `-target` as a means to operate on isolated portions of very
large configurations, prefer instead to break large configurations into
several smaller configurations that can each be independently applied.
[Data sources](/docs/language/data-sources/index.html) can be used to access
information about resources created in other configurations, allowing
a complex system architecture to be broken down into more manageable parts
that can be updated independently.

## Security Warning

Saved plan files (with the `-out` flag) encode the configuration,
state, diff, and _variables_. Variables are often used to store secrets.
Therefore, the plan file can potentially store secrets.

Terraform itself does not encrypt the plan file. It is highly
recommended to encrypt the plan file if you intend to transfer it
or keep it at rest for an extended period of time.

Future versions of Terraform will make plan files more
secure.

## Passing a Different Configuration Directory

Terraform v0.13 and earlier accepted an additional positional argument giving
a directory path, in which case Terraform would use that directory as the root
module instead of the current working directory.

That usage is still supported in Terraform v0.14, but is now deprecated and we
plan to remove it in Terraform v0.15. If your workflow relies on overriding
the root module directory, use
[the `-chdir` global option](./#switching-working-directory-with-chdir)
instead, which works across all commands and makes Terraform consistently look
in the given directory for all files it would normaly read or write in the
current working directory.

If your previous use of this legacy pattern was also relying on Terraform
writing the `.terraform` subdirectory into the current working directory even
though the root module directory was overridden, use
[the `TF_DATA_DIR` environment variable](/docs/cli/config/environment-variables.html#tf_data_dir)
to direct Terraform to write the `.terraform` directory to a location other
than the current working directory.
