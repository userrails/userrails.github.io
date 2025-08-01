---
layout: post
title: "The Complete Guide to Terraform"
date: 2025-07-16 4:58:00 +0545
categories: [cloud devops aws gcp azure docker terraform]
tags: [aws, gcp, azure, cloud, devops, deployment, docker, terraform]
---

# Getting Started with Terraform: Infrastructure as Code Made Simple

## Introduction

Terraform is a powerful Infrastructure as Code (IaC) tool developed by HashiCorp that allows developers and DevOps engineers to define, provision, and manage infrastructure resources using declarative configuration files. Instead of manually clicking through cloud consoles or writing complex scripts, Terraform enables you to describe your desired infrastructure state in human-readable configuration files and automatically creates, updates, or destroys resources to match that state.

Think of Terraform as a blueprint for your infrastructure. Just as an architect creates blueprints before building a house, Terraform lets you define your infrastructure before deploying it. This approach brings software development best practices to infrastructure management, making it more reliable, repeatable, and maintainable.

## What is Infrastructure as Code?

Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable configuration files, rather than through physical hardware configuration or interactive configuration tools. With IaC, you can:

- **Version control your infrastructure** - Track changes, rollback if needed
- **Reproduce environments** - Create identical dev, staging, and production environments
- **Collaborate effectively** - Share infrastructure configurations with team members
- **Automate deployments** - Integrate with CI/CD pipelines

## How Terraform Works

Terraform follows a simple workflow:

1. **Write** - Define infrastructure in `.tf` configuration files
2. **Plan** - Preview changes before applying them
3. **Apply** - Create, update, or destroy infrastructure
4. **Manage** - Track infrastructure state and manage lifecycle

### Key Concepts

- **Providers** - Plugins that interact with APIs (AWS, Azure, GCP, Docker, etc.)
- **Resources** - Infrastructure components (servers, databases, networks)
- **State** - Terraform's record of managed infrastructure
- **Modules** - Reusable configuration packages

## Usage Examples

### Basic Example: Docker Container

Here's a simple example that creates an nginx container using Docker:

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "web" {
  image = docker_image.nginx.name
  name  = "terraform-nginx"
  ports {
    internal = 80
    external = 8080
  }
}
```

* You might need to modify existing configuration with correct docker host path.


```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

provider "docker" {
  host = "unix:///Users/siv/.docker/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "web" {
  image = docker_image.nginx.name
  name  = "terraform-nginx"
  ports {
    internal = 80
    external = 8080
  }
}

```

* Run following command on each config edit. Make sure your Docker is restarted/stopped-started

```
terraform init
# Terraform has been successfully initialized!

terraform validate
# Success! The configuration is valid.

terraform apply
# Apply complete! Resources: 1 added, 1 changed, 0 destroyed.

terraform destroy
# Destroy complete! Resources: 2 destroyed.
```

* Possible error looks like:

```
Error: Error pinging Docker server, please make sure that unix:///var/run/docker.sock is reachable and has a  '_ping' endpoint. Error: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
│
│   with provider["registry.terraform.io/kreuzwerker/docker"],
│   on main.tf line 9, in provider "docker":
│    9: provider "docker" {}
```

* See Docker Context

```
docker context ls
docker context use desktop-linux
```

```
NAME              DESCRIPTION                               DOCKER ENDPOINT                             ERROR
default           Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
desktop-linux *   Docker Desktop                            unix:///Users/siv/.docker/run/docker.sock
```



### AWS EC2 Instance Example

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  
  tags = {
    Name = "MyWebServer"
  }
}
```

### Essential Commands

```bash
# Initialize Terraform (download providers)
terraform init

# Check configuration syntax
terraform validate

# Preview changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy

# Show current state
terraform show

# List managed resources
terraform state list
```

## Pros and Cons

### Advantages

**🎯 Declarative Approach**
- Describe what you want, not how to build it
- Terraform figures out the steps to reach desired state

**🔄 Idempotent Operations**
- Safe to run multiple times
- Only makes necessary changes

**☁️ Multi-Cloud Support**
- Works with AWS, Azure, GCP, and 100+ providers
- Avoid vendor lock-in

**📋 State Management**
- Tracks current infrastructure state
- Enables safe updates and rollbacks

**🔒 Plan Before Apply**
- Preview changes before execution
- Reduces risk of unintended modifications

**📦 Modularity**
- Reusable modules for common patterns
- Promote best practices and consistency

**👥 Team Collaboration**
- Version control integration
- Consistent environments across teams

### Disadvantages

**📚 Learning Curve**
- HCL syntax to master
- Understanding of underlying cloud services required

**🗃️ State File Management**
- Critical state file needs secure storage
- Corruption can cause issues

**🔄 Provider Dependencies**
- Relies on third-party providers
- Updates may introduce breaking changes

**🔧 Limited Logic**
- Not a full programming language
- Complex conditional logic can be challenging

**💰 Cost Considerations**
- Easy to accidentally create expensive resources
- Need proper cost monitoring

**🐛 Debugging Challenges**
- Error messages can be cryptic
- Troubleshooting requires deep understanding

## Deployment Options

### 1. Local Development

**Best for:** Learning, small projects, testing

```bash
# Simple local workflow
terraform init
terraform plan
terraform apply
```

**Considerations:**
- State stored locally
- No collaboration features
- Good for experimentation

### 2. Remote State with Cloud Storage

**Best for:** Team collaboration, production environments

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
  }
}
```

**Benefits:**
- Shared state across team
- State locking prevents conflicts
- Backup and versioning

### 3. Terraform Cloud

**Best for:** Enterprise teams, advanced workflows

**Features:**
- Remote execution
- Private module registry
- Policy as code
- Cost estimation
- Team management

### 4. CI/CD Integration

**Best for:** Automated deployments

```yaml
# GitHub Actions example
name: Terraform
on: [push]
jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
      - name: Terraform Plan
        run: terraform plan
      - name: Terraform Apply
        run: terraform apply -auto-approve
```

### 5. Multi-Environment Setup

**Best for:** Managing dev/staging/prod environments

```
project/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── global/
```

## Best Practices

### 1. Project Structure
- Use modules for reusable components
- Separate environments
- Keep configurations DRY (Don't Repeat Yourself)

### 2. State Management
- Use remote state for team projects
- Enable state locking
- Regular state backups

### 3. Security
- Never commit sensitive data
- Use variables for secrets
- Implement proper IAM policies

### 4. Version Control
- Tag releases
- Use meaningful commit messages
- Review changes before merging

### 5. Testing
- Validate configurations regularly
- Test in lower environments first
- Use terraform plan extensively

## Common Use Cases

### 1. Cloud Migration
- Migrate existing infrastructure to cloud
- Ensure consistency across environments
- Gradual migration strategies

### 2. Disaster Recovery
- Quickly recreate infrastructure
- Multi-region deployments
- Automated backup strategies

### 3. Development Environments
- Spin up/down dev environments
- Consistent developer setups
- Cost optimization

### 4. Compliance and Governance
- Standardized configurations
- Policy enforcement
- Audit trails

## Getting Started Tips

1. **Start Small** - Begin with simple resources like Docker containers
2. **Use Official Providers** - Stick to verified providers from HashiCorp
3. **Read Documentation** - Provider docs are your best friend
4. **Practice Locally** - Use Docker or local providers for learning
5. **Join Community** - Engage with Terraform community for support

## Conclusion

Terraform has revolutionized how we approach infrastructure management by bringing software development practices to infrastructure provisioning. Its declarative approach, multi-cloud support, and strong ecosystem make it an essential tool for modern DevOps practices.

While there's a learning curve, the benefits of using Terraform far outweigh the initial investment in time and effort. Whether you're managing a single server or a complex multi-cloud architecture, Terraform provides the tools and flexibility needed to manage infrastructure efficiently and reliably.
---

## Error and Fixes

![Terraform]({{ "../assets/img/terraform/terraform.png" | absolute_url }})

# Rails + Nginx + Terraform Docker Setup

## Problem
I had a Rails app running in Docker at `localhost:3000` and an nginx container managed by Terraform at `localhost:8080`. I wanted to serve the Rails app through nginx instead of having separate ports.

## Ways to Do: Multiple Containers vs Single Container
Initially, I was confused why I couldn't run Rails **inside** the nginx container like traditional deployments:

**Traditional Server (what I expected):**

```
Server
├── nginx (web server)
├── Rails app (process)
└── Database
```

**Docker Way (what actually happens):**

```
Container 1: nginx
Container 2: Rails app
Container 3: Database
```

**Key insight:** Docker follows "one process per container" principle. Containers communicate over Docker's internal network, not as processes in the same system.

## Solution: Nginx Reverse Proxy

### 1. Create nginx.conf

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://host.docker.internal:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 2. Update Terraform main.tf

```hcl
resource "docker_image" "nginx" {
  name = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "web" {
  image = docker_image.nginx.name
  name  = "terraform-nginx"
  ports {
    internal = 80
    external = 8080
  }

  volumes {
    host_path = "${path.cwd}/nginx.conf"
    container_path = "/etc/nginx/conf.d/default.conf"
  }
}
```

### 3. Test and Apply

```bash
# Test nginx config syntax
docker run --rm -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf nginx nginx -t

# Apply changes
terraform apply

# Test the setup
curl http://localhost:8080
```

## Result
- `localhost:8080` now serves Rails app through nginx
- nginx acts as reverse proxy to Rails container
- Clean separation of concerns with Docker containers

{% include inarticle-adsense.html %}