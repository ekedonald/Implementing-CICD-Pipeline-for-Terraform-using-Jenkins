# Implementing CICD Pipeline for Terraform using Jenkins

## Prerequisites
* Download the docker on your computer, ensure you download the right Docker release that is compatitble with the OS of your computer.

## How to Implement CICD Pipeline for Terraform using Jenkins
The following steps are taken to implement a CICD pipeline for Terraform using Jenkins:

### Step 1: Setting Up The Environment
Jenkins comes with a docker image that can be used out of the box to run a container with all the relevant dependencies for Jenkins. But because we have a unique requirement to run terraform, we need to find a way to extend the readily available jenkins image. To acheive this, we must write a `dockerfile` and include the necessary dependencies to implement this project as shown below:

1. Create a directory and name it `terraform-with-cicd`
2. Create a file and name it `Dockerfile`
3. Copy and paste the content into the `Dockerfile`
```sh
 # Use the official Jenkins base image
 FROM jenkins/jenkins:lts

 # Switch to the root user to install additional packages
 USER root

 # Install necessary tools and dependencies (e.g., Git, unzip, wget, software-properties-common)
 RUN apt-get update && apt-get install -y \
     git \
     unzip \
     wget \
     software-properties-common \
     && rm -rf /var/lib/apt/lists/*

 # Install Terraform
 RUN apt-get update && apt-get install -y gnupg software-properties-common wget \
     && wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg \
     && gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint \
     && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
     && apt-get update && apt-get install -y terraform \
     && rm -rf /var/lib/apt/lists/*

 # Set the working directory
 WORKDIR /app

 # Print Terraform version to verify installation
 RUN terraform --version

 # Switch back to the Jenkins user
 USER jenkins
```

#### Explaining the Dockerfile
1. `FROM jenkins/jenkins:lts`: This line specifies the base image for the Dockerfile. In this case, it's using the official Jenkins LTS (Long Term Support) image as a starting point.

2. `USER root`: This command switches to the root user within the Docker image. This is done to perform actions that require elevated permissions such as installing packages.

3. Install necessary tools and dependencies (i.e. Git, unzip, wget, software-properties-common). The `apt-get update` command refreshes the package list and `apt get install` installs the specifies packages `(i.e. git, unzip, wget, software-properties-common)`. The `&&` is used to chain commands and `rm -rf /var/lib/apt/lists/*` removes unnecessary package lists helping to reduce the size of the Docker image.
```sh
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    wget \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*
```
