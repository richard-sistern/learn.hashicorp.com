# HashiCorp Learn
Notes and examples from the HashiCorp [tutorials](https://learn.hashicorp.com/) site covering the following technology:

- [Terraform](#Terraform)
- [Vault](#Vault)
- [Consul](#Vault)
- Vagrant
- [Packer](#Packer)

Some parts are Windows specific, the HasiCorp tutorials have instructions for different operating systems.

## Bookmark

https://learn.hashicorp.com/tutorials/packer/getting-started-build-image?in=packer/getting-started#a-windows-example

## Terraform

### Install
Download platform specific version of [Terraform](https://www.terraform.io/downloads.html) from HashiCorp

Copy .exe to c:\hashicorp

Add the directory to PATH

```powershell
# Backup current PATH (just in case!)
[Environment]::GetEnvironmentVariable('Path', 'Machine') > PATH.txt

# Add directory to PATH
# Requires elevated rights
$path = [Environment]::GetEnvironmentVariable('Path', 'Machine') + ';c:\hashicorp'
[Environment]::SetEnvironmentVariable("Path", $path, 'Machine')
```

Test with

```sh
terraform --version
```

### Docker Demo

Install [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install)

Create a folder for the demo

```shell
mkdir terraform-docker-demo && cd terraform-docker-demo
```

Create file to hold the configuration

```powershell
New-Item main.tf -Type File
```

Copy the following into main.tf

```hcl
terraform {
  required_providers {
    docker = {
      source = "terraform-providers/docker"
    }
  }
}

provider "docker" {
  host    = "npipe:////.//pipe//docker_engine"
}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

Initialise the project (downloads Docker provider)

```shell
terraform init
```

Provision the demo container

```powershell
terraform apply
```

Test the container

```powershell
(iwr http://localhost:8000).Content
```

```shell
docker ps
```

Remove the container

```shell
terraform destroy
```

### AWS Demo

Terraform requires the following to work with AWS:

- An AWS account
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) 
- AWS [credentials](https://console.aws.amazon.com/iam/home?#security_credential) configured with `aws configure`

This creates a file at `~/.aws/credentials` on MacOS and Linux or `%UserProfile%\.aws\credentials` on Windows which is used to store your credentials.

Create a demo folder and file

```shell
mkdir terraform-aws-demo && cd terraform-aws-demo
New-Item aws.tf -Type File
```

#### Create

Add the following to aws.tf

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 2.70"
    }
  }
}

provider "aws" {
  profile = "default"
  region  = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"
}
```

Not necessarily required, but good practice in a team

```shell
terraform fmt

terraform validate
```

Apply changes to AWS

```shell
terrform apply
```

Validate in EC2 console and inspect the current state using

```shell
terraform show

terraform state list
```

#### Change

Modify the .tf to change the AMI to Ubuntu 16.04

```hcl
resource "aws_instance" "example" { 
	- ami           = "ami-830c94e3" 
	+ ami           = "ami-08d70e59c07c61a3a"  
	instance_type = "t2.micro" 
}
```

Terraform will know to destroy the old one when deployed with 

```shell
terraform apply
```

The output contains the following:

- `+/-` means Terraform will destroy and recreate the resource
- `~` means Terraform will update the resource in place

#### Destroy 

Best to avoid spending **$$$**

```shell
terraform destroy
```

The `terraform destroy` command terminates any and all resources defined in the configuration file.  Use with caution!

#### Input Variables

Create a `variables.tf` file and enter the following

```hcl
variable "region" {
  default = "us-west-2"
}
```

> The file name is unimportant.  Terraform will load all .tf files in a directory

Change the region in  to use the new variable

```hcl
provider "aws" {
  profile = "default"
  # region  = "us-west-2"
  region = var.region
}
```

Other methods of assigning variables (preference order)

##### Command Line Flags

terraform apply -var 'region=us-west-2'

##### File

Create a `terraform.tfvars` file to store the variables

```hcl
region = "us-west-2"
```

Additional files can be specified at run-time

```shell
terraform apply -var-file="secret.tfvars"  -var-file="production.tfvars"
```

*Note: this could be used for staging/production to use the same Terraform configuration*

##### Environment Variables

Terraform can read environment variables in the format `TF_VAR_name`, for example `TF_VAR_region`

##### UI

Terraform will request any missing variables when running 

##### Defaults

If no value is assigned, the specified default value will be used

#### Query Data with Outputs

Ensure environment is ready

```bash
terraform init
terraform apply
```

##### Output EC2 instance configuration

Create a file called `outputs.tf`

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.example.id
}

output "instance_public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.example.public_ip
}
```

```bash
terraform apply

terraform output
instance_id = "i-095e3a5b5432873d4"
instance_public_ip = "54.190.181.203"
```

Tutorial: [Output data from Terraform](https://learn.hashicorp.com/tutorials/terraform/outputs?in=terraform/configuration-language)

#### Store Remote State

Configure Terraform configuration to store remote state

```hcl
backend "remote" {
  organization = "<ORG_NAME>"
  workspaces {
  	name = "Example-Workspace"
  }
}
```

In Work Space > Settings > General

Change the `execution mode` to `local`

Login to Terraform Cloud

```bash
terraform login
terraform init
```

Terraform is now storing state remotely

## Vault


## Consul

## Packer

*Notes from HashiCorp [getting started](https://learn.hashicorp.com/collections/packer/getting-started) tutorial*

Verify install with `packer version`

### Build an Image

Create the `example.pkr.hcl` file.  The `source` block configures a specific builder plugin, which is invoked by a `build`.  A `source` can be reused across multiple `builds` and you can use multiple `sources` in a single build.

The `example.pkr.hcl` builds an EBS-backed AMI by launching a source AMI, provisioning on top of that, and re-packaging it into a new AMI. 

Configure AWS access and validate

```bash
export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY

packer validate example.pkr.hcl
```

*In Windows, use `set AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY` or `$env:"AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY"` or create a credentials file in `%USERPROFILE%.aws\credentials` *

The file is valid if the validate command doesn't generate any output.

Build the image by calling `packer build` with the template file

```bash
packer build \
    -var 'ami_name=packer-tutorial' \
    example.pkr.hcl
```

**Packer only builds images, you need to destroy it manually.**

Remove the AMI by first deregistering it on the [AWS AMI management page](https://console.aws.amazon.com/ec2/home?region=us-east-1#s=Images). Next, delete the associated snapshot on the [AWS snapshot management page](https://console.aws.amazon.com/ec2/home?region=us-east-1#s=Snapshots).

[Packer a Complete Guide with Example](https://medium.com/techno101/packer-a-complete-guide-with-example-cf062b7495eb)

