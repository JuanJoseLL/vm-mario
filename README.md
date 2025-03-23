# Azure VM and Super Mario Container Set Up Guide

### Overview

This guide walks through the complete process of:

1. Creating an Ubuntu VM in Azure using Terraform
2. Using Ansible to install Docker on the VM
3. Deploying a Super Mario Bros game container
4. Accessing the game via web browser

### Prerequisites
1. Azure account and subscription
2. Terraform installed locally
3. Ansible installed locally


## Step 1: Deploying the Azure VM with Terraform

### 1.1 Define Resources in Terraform

- **Resource Group:**  
  Creates a resource group based on the specified location and a random prefix.

- **Virtual Network and Subnet:**  
  Establishes a virtual network (`vnet`) with an address space of `10.0.0.0/16` and a subnet (`subnet`) with a range of `10.0.1.0/24`.

- **Public IP and NSG:**  
  Sets up a dynamic public IP and a Network Security Group that allows inbound traffic on ports:
  - **22 (SSH)**
  - **8787 (Custom, for Mario)**

- **Network Interface and Association:**  
  Creates a network interface (NIC) that attaches the public IP and NSG to the VM.

- **Storage Account:**  
  A storage account is created for boot diagnostics.

- **Linux Virtual Machine:**  
  Deploys a VM using the Ubuntu 22.04 LTS image.  
  - The VM is configured with a username (`adminuser`) and password (`adminPassword1234!`).
  - The VM uses the NIC and connects to the subnet.
  - It also includes an extension that runs a custom script to install the Apache2 web server.

- **Random Resources:**  
  Uses random ID and pet resources to generate unique names for the resources.

### 1.2 Execute Terraform

1. **Initialize Terraform:**
   ```bash
   terraform init
   ```
2. **Plan de deployment:**
   ```bash
   terraform plan -out main.tfplan 
   ```
3. **Apply the configuration:**
   ```bash
   terraform apply main.tfplan 
   ```
The resources have been creaeted in the Azure cloud and you can check the VM ip address with this command:

echo $(terraform output -raw public_ip_address)


## Step 2: Configuring the VM with Ansible

### 2.1 Configure the inventory:

  In the hosts.ini file, change the ip adress with the VM'S address 
  
  ```bash
   [azure_vm]
    172.190.231.253 ansible_user=adminuser ansible_ssh_pass=adminPassword1234!
   ```

### 2.2 Ansible Playbook

The provided Ansible playbook contains the following plays:

1. **Docker Installation:**
   - Installs the necessary dependencies for Docker.
   - Adds the official Docker GPG key and repository.
   - Installs Docker CE.

2. **Docker Container Deployment:**
   - Uses the `docker_container` module to run the Mario container.
   - The container image `pengbai/docker-supermario:latest` is started.
   - Maps container port `8080` to host port `8787` so the game is accessible from a web browser.

### 2.3 Run the Ansible Playbook

Execute the playbook by running:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install_docker.yml
```

and then:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/run_container.yml
```


## Step 3: Accessing the Mario Container

In the borowser now the VM its accesible through the ip of the VM and the port of the container
```
http://172.190.231.253 :8787
```
And the mario game appears

## Cleanup

When finished, you can destroy the resources created by Terraform:

```bash
terraform destroy
```

This command will remove the Azure VM and all related resources to avoid unnecessary costs.

