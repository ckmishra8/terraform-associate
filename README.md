# [Terraform](https://learn.hashicorp.com/collections/terraform/certification)
## Providers

 - Terraform relies on plugins called "providers" to interact with remote systems.
 - Terraform configurations must declare which providers they require so that Terraform can install and use them.
 - Every resource type is implemented by a provider; without providers, Terraform can't manage any kind of infrastructure.
 - Each Terraform module must declare which providers it requires, so that Terraform can install and use them. Provider requirements are declared in a  `required_providers`  block.
 - A provider requirement consists of a local name, a source location, and a version constraint:
```
terraform {
  required_providers {
    mycloud = {
      source  = "mycorp/mycloud"
      version = "~> 1.0"
    }
  }
}
```
- The `required_providers` block must be nested inside the top-level [`terraform`  block](https://www.terraform.io/docs/configuration/terraform.html) (which can also contain other settings).

**Note:** The `name = { source, version }` syntax for `required_providers` was added in Terraform v0.13. Previous versions of Terraform used a version constraint string instead of an object (like `mycloud = "~> 1.0"`), and had no way to specify provider source addresses. If you want to write a module that works with both Terraform v0.12 and v0.13

- Each provider has two identifiers:
		- A unique  __source address,__  which is only used when requiring a provider.
		- A  __local name,__  which is used everywhere else in a Terraform module.
- A provider's source address is its global identifier. It also specifies the primary location where Terraform can download it.
- Source addresses consist of three parts delimited by slashes (`/`), as follows:
`[<HOSTNAME>/]<NAMESPACE>/<TYPE>`

-   **Hostname**  (optional): The hostname of the Terraform registry that distributes the provider. If omitted, this defaults to  `registry.terraform.io`, the hostname of  [the public Terraform Registry](https://registry.terraform.io/).
- 
-   **Namespace:**  An organizational namespace within the specified registry. For the public Terraform Registry and for Terraform Cloud's private registry, this represents the organization that publishes the provider. This field may have other meanings for other registry hosts.
-   **Type:**  A short name for the platform or system the provider manages. Must be unique within a particular namespace on a particular registry host.
- A provider configuration is created using a  `provider`  block:
```
provider "google" {
  project = "acme-app"
  region  = "us-central1"
}
```
- The name given in the block header (`"google"` in this example) is the [local name](https://www.terraform.io/docs/configuration/provider-requirements.html#local-names) of the provider to configure. This provider should already be included in a `required_providers` block.
- You can optionally define multiple configurations for the same provider, and select which one to use on a per-resource or per-module basis. The primary reason for this is to support multiple regions for a cloud platform; other examples include targeting multiple Docker hosts, multiple Consul hosts, etc.
```
# The default provider configuration; resources that begin with `aws_` will use
# it as the default, and it can be referenced as `aws`.
provider "aws" {
  region = "us-east-1"
}

# Additional provider configuration for west coast region; resources can
# reference this as `aws.west`.
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```
- When Terraform needs the name of a provider configuration, it expects a reference of the form  `<PROVIDER NAME>.<ALIAS>`. In the example above,  `aws.west`  would refer to the provider with the  `us-west-2`  region.
- The dependency lock file is a file that belongs to the configuration as a whole, rather than to each separate module in the configuration.
- The lock file is always named  `.terraform.lock.hcl`, and this name is intended to signify that it is a lock file for various items that Terraform caches in the  `.terraform`  subdirectory of your working directory.
- Terraform automatically creates or updates the dependency lock file each time you run [the  `terraform init`  command](https://www.terraform.io/docs/commands/init.html).

## State
- Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures.
- This state is stored by default in a local file named `terraform.tfstate`, but it can also be stored remotely, which works better in a team environment.
- Prior to any operation, Terraform does a [refresh](https://www.terraform.io/docs/commands/refresh.html) to update the state with the real infrastructure.
- Backends are responsible for storing state and providing an API for  [state locking](https://www.terraform.io/docs/state/locking.html). State locking is optional.
- Terraform starts with a single workspace named "default". This workspace is special both because it is the default and also because it cannot ever be deleted.
- If you manage any sensitive data with Terraform (like database passwords, user passwords, or private keys), treat the state itself as sensitive data.

## Terraform Settings
- The special  `terraform`  configuration block type is used to configure some behaviours of Terraform itself, such as requiring a minimum Terraform version to apply your configuration.
- Terraform settings are gathered together into  `terraform`  blocks:
```
terraform {
  # ...
}
```
- Within a `terraform` block, only constant values can be used.
- The `required_version` setting accepts a [version constraint string,](https://www.terraform.io/docs/configuration/version-constraints.html) which specifies which versions of Terraform can be used with your configuration. If the running version of Terraform doesn't match the constraints specified, Terraform will produce an error and exit without taking any further actions.
- When you use  [child modules](https://www.terraform.io/docs/configuration/blocks/modules/index.html), each module can specify its own version requirements. The requirements of all modules in the tree must be satisfied.
- The  `required_providers`  block specifies all of the providers required by the current module, mapping each local provider name to a source address and a version constraint.
```
terraform {
  required_providers {
    aws = {
      version = ">= 2.7.0"
      source = "hashicorp/aws"
    }
  }
}
```
- In releases where experimental features are available, you can enable them on a per-module basis by setting the  `experiments`  argument inside a  `terraform`  block:
```
terraform {
  experiments = [example]
}
```
## Resources
- _Resources_ are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects.
- The Meta-Arguments section documents special arguments that can be used with every resource type, including [`depends_on`](https://www.terraform.io/docs/configuration/meta-arguments/depends_on.html), [`count`](https://www.terraform.io/docs/configuration/meta-arguments/count.html), [`for_each`](https://www.terraform.io/docs/configuration/meta-arguments/for_each.html), [`provider`](https://www.terraform.io/docs/configuration/meta-arguments/resource-provider.html), and [`lifecycle`](https://www.terraform.io/docs/configuration/meta-arguments/lifecycle.html).
- [Provisioners](https://www.terraform.io/docs/configuration/blocks/resources/provisioners/index.html) documents configuring post-creation actions for a resource using the `provisioner` and `connection` blocks.
```
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```
```
resource "aws_db_instance" "example" {
  # ...

  timeouts {
    create = "60m"
    delete = "2h"
  }
}
```
- Use the  `depends_on`  meta-argument to handle hidden resource or module dependencies that Terraform can't automatically infer.
- Explicitly specifying a dependency is only necessary when a resource or module relies on some other resource's behavior but  _doesn't_  access any of that resource's data in its arguments.
```
resource "aws_iam_role" "example" {
  name = "example"

  # assume_role_policy is omitted for brevity in this example. See the
  # documentation for aws_iam_role for a complete example.
  assume_role_policy = "..."
}

resource "aws_iam_instance_profile" "example" {
  # Because this expression refers to the role, Terraform can infer
  # automatically that the role must be created first.
  role = aws_iam_role.example.name
}

resource "aws_iam_role_policy" "example" {
  name   = "example"
  role   = aws_iam_role.example.name
  policy = jsonencode({
    "Statement" = [{
      # This policy allows software running on the EC2 instance to
      # access the S3 API.
      "Action" = "s3:*",
      "Effect" = "Allow",
    }],
  })
}

resource "aws_instance" "example" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"

  # Terraform can infer from this that the instance profile must
  # be created before the EC2 instance.
  iam_instance_profile = aws_iam_instance_profile.example

  # However, if software running in this EC2 instance needs access
  # to the S3 API in order to boot properly, there is also a "hidden"
  # dependency on the aws_iam_role_policy that Terraform cannot
  # automatically infer, so it must be declared explicitly:
  depends_on = [
    aws_iam_role_policy.example,
  ]
}
```
- By default, a [resource block](https://www.terraform.io/docs/configuration/blocks/resources/syntax.html) configures one real infrastructure object. (Similarly, a [module block](https://www.terraform.io/docs/configuration/blocks/modules/syntax.html) includes a child module's contents into the configuration one time.) However, sometimes you want to manage several similar objects (like a fixed pool of compute instances) without writing a separate block for each one. Terraform has two ways to do this: `count` and [`for_each`](https://www.terraform.io/docs/configuration/meta-arguments/for_each.html).
```
resource "aws_instance" "server" {
  count = 4 # create four similar EC2 instances

  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${count.index}"
  }
}
```
- If your instances are almost identical,  `count`  is appropriate. If some of their arguments need distinct values that can't be directly derived from an integer, it's safer to use  `for_each`.
Map:

```
resource "azurerm_resource_group" "rg" {
  for_each = {
    a_group = "eastus"
    another_group = "westus2"
  }
  name     = each.key
  location = each.value
}

```

Set of strings:

```
resource "aws_iam_user" "the-accounts" {
  for_each = toset( ["Todd", "James", "Alice", "Dottie"] )
  name     = each.key
}

```

Child module:

```
# my_buckets.tf
module "bucket" {
  for_each = toset(["assets", "media"])
  source   = "./publish_bucket"
  name     = "${each.key}_bucket"
}
```
- In blocks where  `for_each`  is set, an additional  `each`  object is available in expressions, so you can modify the configuration of each instance. This object has two attributes:
-   [`each.key`](https://www.terraform.io/docs/configuration/meta-arguments/for_each.html#each-key)  — The map key (or set member) corresponding to this instance.
-   [`each.value`](https://www.terraform.io/docs/configuration/meta-arguments/for_each.html#each-value)  — The map value corresponding to this instance. (If a set was provided, this is the same as  `each.key`.)
- The Terraform language doesn't have a literal syntax for  [set values](https://www.terraform.io/docs/configuration/types.html#collection-types), but you can use the  `toset`  function to explicitly convert a list of strings to a set:
```
locals {
  subnet_ids = toset([
    "subnet-abcdef",
    "subnet-012345",
  ])
}

resource "aws_instance" "server" {
  for_each = local.subnet_ids

  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
  subnet_id     = each.key # note: each.key and each.value are the same for a set

  tags = {
    Name = "Server ${each.key}"
  }
}

```
- The `provider` meta-argument specifies which provider configuration to use for a resource, overriding Terraform's default behavior of selecting one based on the resource type name. Its value should be an unquoted `<PROVIDER>.<ALIAS>` reference.
```
# default configuration
provider "google" {
  region = "us-central1"
}

# alternate configuration, whose alias is "europe"
provider "google" {
  alias  = "europe"
  region = "europe-west1"
}

resource "google_compute_instance" "example" {
  # This "provider" meta-argument selects the google provider
  # configuration whose alias is "europe", rather than the
  # default configuration.
  provider = google.europe

  # ...
}
```
- _Data sources_ allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to make use of information defined outside of Terraform, or defined by another separate Terraform configuration.
```
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

## Provisioners
- Provisioners can be used to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo The server's IP address is ${self.private_ip}"
  }
}
```
- Expressions in  `provisioner`  blocks cannot refer to their parent resource by name. Instead, they can use the special  `self`  object.
- The  `self`  object represents the provisioner's parent resource, and has all of that resource's attributes. For example, use  `self.public_ip`  to reference an  `aws_instance`'s  `public_ip`  attribute.
- By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during _creation_, not during updating or any other lifecycle.
- If a creation-time provisioner fails, the resource is marked as **tainted**.
- A tainted resource will be planned for destruction and recreation upon the next `terraform apply`. Terraform does this because a failed provisioner can leave a resource in a semi-configured state.
- You can change this behaviour by setting the  `on_failure`  attribute, which is covered in detail below:
```
resource "aws_instance" "web" {
  # ...
  provisioner "local-exec" {
    command    = "echo The server's IP address is ${self.private_ip}"
    on_failure = continue/fail
  }
}
```
- If  `when = destroy`  is specified, the provisioner will run when the resource it is defined within is  _destroyed_.
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroy-time provisioner'"
  }
}
```
- Multiple provisioners can be specified within a resource. Multiple provisioners are executed in the order they're defined in the configuration file.
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo first"
  }

  provisioner "local-exec" {
    command = "echo second"
  }
}
```
- The  `file`  provisioner is used to copy files or directories from the machine executing Terraform to the newly created resource. The  `file`  provisioner supports both  `ssh`  and  `winrm`  type  [connections](https://www.terraform.io/docs/provisioners/connection.html).
```
resource "aws_instance" "web" {
  # ...

  # Copies the myapp.conf file to /etc/myapp.conf
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
  }

  # Copies the string in content into /tmp/file.log
  provisioner "file" {
    content     = "ami used: ${self.ami}"
    destination = "/tmp/file.log"
  }

  # Copies the configs.d folder to /etc/configs.d
  provisioner "file" {
    source      = "conf/configs.d"
    destination = "/etc"
  }

  # Copies all files and folders in apps/app1 to D:/IIS/webapp1
  provisioner "file" {
    source      = "apps/app1/"
    destination = "D:/IIS/webapp1"
  }
}
```
- The  `local-exec`  provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource. See the  `remote-exec`  [provisioner](https://www.terraform.io/docs/provisioners/remote-exec.html)  to run commands on the resource.
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo ${aws_instance.web.private_ip} >> private_ips.txt"
  }
}
```
- The `remote-exec` provisioner invokes a script on a remote resource after it is created.
```
resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "puppet apply",
      "consul join ${aws_instance.web.private_ip}",
    ]
  }
}
```
```
resource "aws_instance" "web" {
  # ...

  provisioner "file" {
    source      = "script.sh"
    destination = "/tmp/script.sh"
  }

  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/script.sh",
      "/tmp/script.sh args",
    ]
  }
}
```
## The Core Terraform Workflow
- The core Terraform workflow has three steps:
	-  **Write**  - Author infrastructure as code.
	- **Plan**  - Preview changes before applying.
	- **Apply**  - Provision reproducible infrastructure.

- The `terraform init` command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. It is safe to run this command multiple times.
- Usage:  `terraform init [options]`
- The following options apply to all of (or several of) the initialization steps:
	- [`-input=true`](https://www.terraform.io/docs/commands/init.html#input-true)  Ask for input if necessary. If false, will error if input was required.
	- [`-lock=false`](https://www.terraform.io/docs/commands/init.html#lock-false)  Disable locking of state files during state-related operations.
	- [`-lock-timeout=<duration>`](https://www.terraform.io/docs/commands/init.html#lock-timeout-lt-duration-gt-)  Override the time Terraform will wait to acquire a state lock. The default is  `0s`  (zero seconds), which causes immediate failure if the lock is already held by another process.
	- [`-no-color`](https://www.terraform.io/docs/commands/init.html#no-color)  Disable color codes in the command output.
	- [`-upgrade`](https://www.terraform.io/docs/commands/init.html#upgrade)  Opt to upgrade modules and plugins as part of their respective installation steps. See the sections below for more details.
- The  `terraform get`  command is used to download and update  [modules](https://www.terraform.io/docs/modules/index.html)  mentioned in the root module.
- Usage:  `terraform get [options]`
- The modules are downloaded into a `.terraform` subdirectory of the current working directory. Don't commit this directory to your version control repository.
- The  `get`  command supports the following option:
	- [`-update`](https://www.terraform.io/docs/commands/get.html#update)  - If specified, modules that are already downloaded will be checked for updates and the updates will be downloaded if present.

## Writing and Modifying Terraform Code
- The  `terraform console`  command provides an interactive console for evaluating  [expressions](https://www.terraform.io/docs/configuration/expressions/index.html).
	- Usage:  `terraform console [options]`
- The `terraform fmt` command is used to rewrite Terraform configuration files to a canonical format and style.
- Usage:  `terraform fmt [options] [DIR]`
- By default, `fmt` scans the current directory for configuration files.
- The command-line flags are all optional. The list of available flags are:
	-   [`-list=false`](https://www.terraform.io/docs/commands/fmt.html#list-false)  - Don't list the files containing formatting inconsistencies.
	-   [`-write=false`](https://www.terraform.io/docs/commands/fmt.html#write-false)  - Don't overwrite the input files. (This is implied by  `-check`  or when the input is STDIN.)
	-   [`-diff`](https://www.terraform.io/docs/commands/fmt.html#diff)  - Display diffs of formatting changes
	-   [`-check`](https://www.terraform.io/docs/commands/fmt.html#check)  - Check if the input is formatted. Exit status will be 0 if all input is properly formatted and non-zero otherwise.
	-   [`-recursive`](https://www.terraform.io/docs/commands/fmt.html#recursive)  - Also process files in subdirectories. By default, only the given directory (or current directory) is processed.
- The `terraform validate` command checks that verify whether a configuration is syntactically valid and internally consistent, regardless of any provided variables or existing state.
	- Usage:  `terraform validate [options]`
	- This command accepts the following options:
		-   [`-json`](https://www.terraform.io/docs/commands/validate.html#json)  - Produce output in a machine-readable JSON format, suitable for use in text editor integrations and other automated systems. Always disables color.
		-   [`-no-color`](https://www.terraform.io/docs/commands/validate.html#no-color)  - If specified, output won't contain any color.

## Provisioning Infrastructure with Terraform
- The `terraform plan` command evaluates a Terraform configuration to determine the desired state of all the resources it declares, then compares that desired state to the real infrastructure objects being managed with the current working directory and workspace.
- The `terraform apply` command performs a plan just like `terraform plan` does, but then actually carries out the planned changes to each resource using the relevant infrastructure provider's API.
- The `terraform destroy` command destroys all of the resources being managed by the current working directory and workspace.

## Command: taint
- The  `terraform taint`  command manually marks a Terraform-managed resource as tainted, forcing it to be destroyed and recreated on the next apply.
- This command _will not_ modify infrastructure, but does modify the state file in order to mark a resource as tainted. Once a resource is marked as tainted, the next [plan](https://www.terraform.io/docs/commands/plan.html) will show that the resource will be destroyed and recreated and the next [apply](https://www.terraform.io/docs/commands/apply.html) will implement this change.
```
$ terraform taint aws_security_group.allow_all
The resource aws_security_group.allow_all in the module root has been marked as tainted.
```
## State Command
- The `terraform state` command is used for advanced state management. As your Terraform usage becomes more advanced, there are some cases where you may need to modify the [Terraform state](https://www.terraform.io/docs/state/index.html). Rather than modify the state directly, the `terraform state` commands can be used in many cases instead.

## Workspaces
- Each Terraform configuration has an associated [backend](https://www.terraform.io/docs/backends/index.html) that defines how operations are executed and where persistent data such as [the Terraform state](https://www.terraform.io/docs/state/purpose.html) are stored.
- The persistent data stored in the backend belongs to a  _workspace_. Initially the backend has only one workspace, called "default", and thus there is only one Terraform state associated with that configuration.
```
$ terraform workspace new bar
Created and switched to workspace "bar"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```
-  Within your Terraform configuration, you may include the name of the current workspace using the `${terraform.workspace}` interpolation sequence. This can be used anywhere interpolations are allowed.
```
resource "aws_instance" "example" {
  count = "${terraform.workspace == "default" ? 5 : 1}"

  # ... other arguments
}
```
```
resource "aws_instance" "example" {
  tags = {
    Name = "web - ${terraform.workspace}"
  }

  # ... other arguments
}
```
## Import
- Terraform is able to import existing infrastructure. This allows you take resources you've created by some other means and bring it under Terraform management.
	- Usage: `terraform import [options] ADDRESS ID`
```
$ terraform import aws_instance.foo i-abcd1234
```
- The example below will import an AWS instance into the  `aws_instance`  resource named  `bar`  into a module named  `foo`:
```
$ terraform import module.foo.aws_instance.bar i-abcd1234
```
## Modules
- Here are some of the ways that modules help solve the problems listed above:
	- Organize configuration
	- Encapsulate configuration
	- Re-use configuration
	- Provide consistency and ensure best practices
	- Using modules can help reduce these errors.
- A Terraform module is a set of Terraform configuration files in a single directory. Even a simple configuration consisting of a single directory with one or more `.tf` files is a module. When you run Terraform commands directly from such a directory, it is considered the **root module**. So in this sense, every Terraform configuration is part of a module. You may have a simple set of Terraform configuration files such as:
```
$ tree minimal-module/
.
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```
- Modules can either be loaded from the local filesystem, or a remote source. Terraform supports a variety of remote sources, including the Terraform Registry, most version control systems, HTTP URLs, and Terraform Cloud or Terraform Enterprise private module registries.
```
# Terraform configuration

terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.21.0"

  name = var.vpc_name
  cidr = var.vpc_cidr

  azs             = var.vpc_azs
  private_subnets = var.vpc_private_subnets
  public_subnets  = var.vpc_public_subnets

  enable_nat_gateway = var.vpc_enable_nat_gateway

  tags = var.vpc_tags
}

module "ec2_instances" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "2.12.0"

  name           = "my-ec2-cluster"
  instance_count = 2

  ami                    = "ami-0c5204531f799e0c6"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [module.vpc.default_security_group_id]
  subnet_id              = module.vpc.public_subnets[0]

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```
- Modules also have output values, which are defined within the module with the `output` keyword. You can access them by referring to `module.<MODULE NAME>.<OUTPUT NAME>`.
- Inside your configuration's directory,  `outputs.tf`  will need to contain:
```
output "vpc_public_subnets" {
  description = "IDs of the VPC's public subnets"
  value       = module.vpc.public_subnets
}

output "ec2_instance_public_ips" {
  description = "Public IP addresses of EC2 instances"
  value       = module.ec2_instances.public_ip
}
```
- When using a new module for the first time, you must run either `terraform init` or `terraform get` to install the module. When either of these commands are run, Terraform will install any new modules in the `.terraform/modules` directory within your configuration's working directory. For local modules, Terraform will create a symlink to the module's directory. Because of this, any changes to local modules will be effective immediately, without having to re-run `terraform get`.
- After following this tutorial, your  `.terraform/modules`  directory will look something like this:
```
.terraform/modules
├── ec2_instances
│   └── terraform-aws-modules-terraform-aws-ec2-instance-ed6dcd9
├── modules.json
└── vpc
    └── terraform-aws-modules-terraform-aws-vpc-2417f60
```
- The syntax for specifying a registry module is `<NAMESPACE>/<NAME>/<PROVIDER>`. For example: `hashicorp/consul/aws`.

### Private Registry Module Sources
- Private registry modules have source strings of the form  `<HOSTNAME>/<NAMESPACE>/<NAME>/<PROVIDER>`. This is the same format as the public registry, but with an added hostname prefix.
```
module "vpc" {
  source = "app.terraform.io/example_corp/vpc/aws"
  version = "0.9.3"
}
```
## Input Variables
- The supported type keywords are:

-   [`string`](https://www.terraform.io/docs/configuration/variables.html#string)
-   [`number`](https://www.terraform.io/docs/configuration/variables.html#number)
-   [`bool`](https://www.terraform.io/docs/configuration/variables.html#bool)

The type constructors allow you to specify complex types such as collections:

-   [`list(<TYPE>)`](https://www.terraform.io/docs/configuration/variables.html#list-lt-type-gt-)
-   [`set(<TYPE>)`](https://www.terraform.io/docs/configuration/variables.html#set-lt-type-gt-)
-   [`map(<TYPE>)`](https://www.terraform.io/docs/configuration/variables.html#map-lt-type-gt-)
-   [`object({<ATTR NAME> = <TYPE>, ... })`](https://www.terraform.io/docs/configuration/variables.html#object-lt-attr-name-gt-lt-type-gt-)
-   [`tuple([<TYPE>, ...])`](https://www.terraform.io/docs/configuration/variables.html#tuple-lt-type-gt-)
- The keyword `any` may be used to indicate that any type is acceptable.

## Output Values
```
output "instance_ip_addr" {
  value       = aws_instance.server.private_ip
  description = "The private IP address of the main server instance."
}
```
```
output "db_password" {
  value       = aws_db_instance.db.password
  description = "The password for logging in to the database."
  sensitive   = true
}
```
- Sensitive output values are still recorded in the  [state](https://www.terraform.io/docs/state/index.html), and so will be visible to anyone who is able to access the state data. For more information, see  [_Sensitive Data in State_](https://www.terraform.io/docs/state/sensitive-data.html).
```
output "instance_ip_addr" {
  value       = aws_instance.server.private_ip
  description = "The private IP address of the main server instance."

  depends_on = [
    # Security group rule must be created before this IP address could
    # actually be used, otherwise the services will be unreachable.
    aws_security_group_rule.local_access,
  ]
}
```
## Local Values
- A local value assigns a name to an  [expression](https://www.terraform.io/docs/configuration/expressions/index.html), so you can use it multiple times within a module without repeating it.
```
locals {
  # Ids for multiple sets of EC2 instances, merged together
  instance_ids = concat(aws_instance.blue.*.id, aws_instance.green.*.id)
}

locals {
  # Common tags to be assigned to all resources
  common_tags = {
    Service = local.service_name
    Owner   = local.owner
  }
}
```
- Once a local value is declared, you can reference it in  [expressions](https://www.terraform.io/docs/configuration/expressions/index.html)  as  `local.<NAME>`.
```
resource "aws_instance" "example" {
  # ...

  tags = local.common_tags
}
```
## Data Sources
- _Data sources_ allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to make use of information defined outside of Terraform, or defined by another separate Terraform configuration.
- Each  [provider](https://www.terraform.io/docs/configuration/providers.html)  may offer data sources alongside its set of  [resource](https://www.terraform.io/docs/configuration/blocks/resources/index.html)  types.
- A data source is accessed via a special kind of resource known as a  _data resource_, declared using a  `data`  block:
```
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```
- Each data instance will export one or more attributes, which can be used in other resources as reference expressions of the form  `data.<TYPE>.<NAME>.<ATTRIBUTE>`. For example:
```
resource "aws_instance" "web" {
  ami           = data.aws_ami.web.id
  instance_type = "t1.micro"
}
```
## Resource Addressing
A  **Resource Address**  is a string that references a specific resource in a larger infrastructure. An address is made up of two parts:
```
[module path][resource spec]
```
- A module path addresses a module within the tree of modules. It takes the form:
```
module.module_name[module index]
```
- An example of the  `module`  keyword delineating between two modules that have multiple instances:
```
module.foo[0].module.bar["a"]
```
- A resource spec addresses a specific resource in the config. It takes the form:
```
resource_type.resource_name[resource index]
```
## Manage explicit dependencies
```
resource "aws_s3_bucket" "example" {
  acl    = "private"
}

resource "aws_instance" "example_c" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"

  depends_on = [aws_s3_bucket.example]
}

module "example_sqs_queue" {
  source  = "terraform-aws-modules/sqs/aws"
  version = "2.1.0"

  depends_on = [aws_s3_bucket.example, aws_instance.example_c]
}
```
