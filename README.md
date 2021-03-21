# HashiCorp Learn
Notes and examples from the HashiCorp [tutorials](https://learn.hashicorp.com/) site covering the following technology:

- [Terraform](#Terraform)
- [Vault](#Vault)
- [Consul](#Vault)
- Vagrant
- Packer

Some parts are Windows specific, the HasiCorp tutorials have instructions for different operating systems.

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

#### Output Variables

Ensure environment is ready

```bash
terraform init
terraform apply
```



## Vault


## Consul

