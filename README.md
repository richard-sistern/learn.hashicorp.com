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



## Vault


## Consul

