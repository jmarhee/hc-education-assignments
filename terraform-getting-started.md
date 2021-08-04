# Getting Started with Terraform

Terraform is a tool that lets you define and provision infrastructure using code. This is also known as IaC. You can use Terraform to define how your environment will look once deployed, create an execution plan, and Terraform will apply it. You can use "providers" to define as code, and manage, resources on many cloud platforms and for a number of other popular tools.

## Installing Terraform

You can visit [Terraform.io](https://www.terraform.io/downloads.html) to download the Terraform command-line tool. 

You will need to decompress this downloaded zipfile, and copy the `terraform` file into your command-line `PATH`.

If you use a package manager like Chocolatey on Windows, or Homebrew on MacOS, Terraform can be installed using the package manager as well. 

## Writing Terraform Plans

You will start by creating and entering your Terraform project directory.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Next, you should create an empty file. This file will contain all of your infrastructure code.

```shell
$ touch main.tf
```

We recommend starting your Terraform file by adding the following to your file:

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
```

These lines are your provider requirements for Terraform, and options for how Terraform will configure each provider. This file will have told Terraform that the Docker provider is required for this project, and then told the provider how to connect to Docker. 

Next, you can add a Docker-provider resource such as a Docker container and a Docker image to your file.

```hcl
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

You may notice that these resources can reference each other. Each resource can be accessed by another resource in your file using `resource_type.name` syntax as you did for `image = docker_image.nginx.latest`. Terraform also allows you to use variables and a limited set of conditional statements in more complex execution plans.

## Initializing Terraform

In order to download the providers you required, and create the resources you defined, you must initialize your Terraform project.

```shell
$ terraform init
```

The output from this command will show Terraform's initializing of your file's provider and which version of each provider or dependency was downloaded. 

```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.14.0...
- Installed kreuzwerker/docker v2.14.0 (self-signed, key ID 24E54F214569A8A5)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, we recommend adding version constraints in a required_providers block
in your configuration, with the constraint strings suggested below.

* kreuzwerker/docker: version = "~> 2.14.0"

Terraform has been successfully initialized!
```

## Managing Infrastructure with Terraform

If Terraform is successfully initialized, next you may view your execution plan before any resources are created.

```bash
$ terraform plan
```

Terraform will display the expected plan for your infrastructure based on your Terraform file.

```bash
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

Terraform will show you that the execution plan it will apply has the container and image resources you added to your `main.tf` file earlier. 

When Terraform prints your execution plan, if it looks as you expected then you can apply your file.

```shell
$ terraform apply
```
Terraform may take a few minutes to run and will display progress as each resource is created. 

```bash
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 0s [id=sha256:62d49f9bab67f7c70ac3395855bf01389eb3175b374e621f6f191bf31b54cd5bnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 3s [id=aabf91fe7344683d9f746169fdf70aa4d0099fc109c923a8d3c30967bfcc1026]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed
```
Terraform will print a message for each resource's status along with an ID for that resource. 

## Updating Plans and Destroying Infrastructure

Once your file is applied, you can make changes to `main.tf` and apply them again if you need to make chances. You can also destroy these resources using the destroy subcommand.

```shell
$ terraform destroy
```

and the output will show you which resources were destroyed.

```bash
docker_image.nginx: Refreshing state... [id=sha256:62d49f9bab67f7c70ac3395855bf01389eb3175b374e621f6f191bf31b54cd5bnginx:latest]
docker_container.nginx: Refreshing state... [id=aabf91fe7344683d9f746169fdf70aa4d0099fc109c923a8d3c30967bfcc1026]
docker_container.nginx: Destroying... [id=aabf91fe7344683d9f746169fdf70aa4d0099fc109c923a8d3c30967bfcc1026]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:62d49f9bab67f7c70ac3395855bf01389eb3175b374e621f6f191bf31b54cd5bnginx:latest]
docker_image.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```
