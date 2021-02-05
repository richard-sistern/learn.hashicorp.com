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



Teardown (avoid **$$$**)

```shell
terraform destroy
```

The `terraform destroy` command terminates any and all resources defined in the configuration file.  Use with caution!

## Vault


## Consul

