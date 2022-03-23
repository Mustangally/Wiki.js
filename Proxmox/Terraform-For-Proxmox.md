---
title: Automating Proxmox VM Deployments with Terraform
description: 
published: true
date: 2022-03-23T16:18:02.105Z
tags: 
editor: markdown
dateCreated: 2022-03-23T16:16:54.076Z
---

In my last post about building Proxmox VM Templates with Cloud-Init I discussed how we can make the process of creating VMs significantly faster and easier. But what does the next step look like? You could choose to create a script and run it each time you want to create a new Template or VM. This works well for small environments, but it doesn't scale well and it doesn't provide much visibility into your infrastructure. Instead, we can use Terraform to create "Infrastructure as Code" to manage the infrastructure and keep track of changes.

## What is Terraform?
Per Hasicorp's own description, "Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files." 
Ok cool. What does that mean? Well what this offers is the ability to turn your infrastructure (or Proxmox Cluster) into description files using code. HCL (Hashicorp Configuration Language) allows us to create declarative definition files that we can track and control over time to keep our infrastructure under control, and provision new VMs or LXC Containers.

## Install Terraform
Installing Terraform on your dev machine is pretty simple. I'll be using Ubuntu 20.04 for this guide, but consult the docs if you're on something different.
Lets start by installing dependencies, then adding the key and repo from Hashicorp, update apt, and install:
```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

## Authentication Methods
Terraform needs to be able to authenticate to your infrastructure in some way. When using Proxmox, we have two options:1. Username/Password - you can use root, or preferably you can create a new Service Account and assign the minimum permissions to complete the needed tasks.2. API Keys - Again you should create a Service Account, but you can generate API keys for the SA for Terraform to use for authentication. We'll walk through this option below.

We're going to have these authentication methods sitting in configuration files, and we don't really want to be storing passwords that way, especially not the root password. Secondly, we'll use a different SA so that we can restrict access just incase somehow our API key was leaked or compromised, without impacting the root account.

### Create the SA and API Key
To add a new user go to Datacenter in the left tab, then Permissions -> Users -> Click add, name the user and click add.

Now that we have a service account, we need to create an API Key to go along with it. The SA currently doesn't have a password, so it has no way to authenticate to the cluster. To do this, click on "API Tokens" below users in the permissions tab, and again click "Add". Select the user, and create your own token ID. Make sure to UNCHECK "Privilege Separation".


Now we can assign a Role to the user so that the SA has permissions to actually effect change. Click on the "Permissions" tab, then "Add":

I'll be using the PVEAdmin Role as it gives mostly unrestricted access. You can see a detailed view of each role in the "Roles" tab.
You'll also need to give your SA permission to the datastore you intend to use. In my case this will be /storage/SSD. Do the same as above to add the permission:

Now that the Proxmox side of things is complete, we can move on to working with Terraform!

# Terraform Setup
When setting up Terraform, you have three different broad level tasks or commands that you will work with. These are "Init", "Plan", and "Apply".
We're going to start with the "Plans". These are essentially config files detailing what you want Terraform to do. Compare these to playbooks for Ansible, if you're familiar with that. These plans will be stored in a directory structure, so lets start by creating a new directory for this: mkdir terraform-example and cd terraform-example
Now we can create the two initial files we're going to work with. These will be `main.tf` and `vars.tf`. The names are fairly self explanatory, but the bulk of our config will be in the `main.tf` file, but it will reference some variables in `vars.tf`. This isn't technically necessary, but it is best practice as it makes management of the config easier down the line.

### Main File
The `main.tf` file is going to hold our configuration for Terraform, basically to tell it what to connect to, and where that system is. To do this we need to add a "provider". This tells Terraform what it's connecting to like AWS, GCP, or in our case Proxmox. The provider that we're going to use is "Telmate Proxmox provider", and adding it is fairly simple:
```
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.9.6" #or whatever the current version is
    }
  }
}
```

### Initialize the Plan
Now that we have a basic plan that allows Terraform to pull the provider, we can run the plan to initialize Terraform to use the Proxmox provider. Simply run `terraform init` from the directory holding your `main.tf` file and watch the output.
```
terraform init

Initializing the backend...
Initializing provider plugins...
- Finding telmate/proxmox versions matching "2.9.6"...
- Installing telmate/proxmox v2.9.6...
- Installed telmate/proxmox v2.9.6 (self-signed, key ID A9EBBE091B35AFCE)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other commands will detect it and remind you to do so if necessary.
```

## Develop an actual Terraform Plan
Now that the provider is installed and Terraform is configured to utilize Proxmox, we can develop the plan further to connect to our Proxmox cluster so that we can actually perform actions. We're going to add on to our `main.tf` and pass through all of the parameters to connect to the server.
```
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.9.6" #or whatever the current version is
    }
  }
}

#this is where we can authenticate to the server
  provider "proxmox" {
    pm_api_url = "https://proxmox.local.kkovach.com/api2/json"
    pm_api_token_id = "terraform@pam!terraform_token"
    pm_api_token_secret = "ENTER SECRET HERE"
  }
  
# now we can start to build our VM
  resource "proxmox_vm_qemu" "test_VM" {
    count = 1
    name = "test-VM-${count.index + 1}" #this is going to index the VM created with a number based on a counter variable.
    
    # we're going to start pointing to our vars file for the following attributes
    target_node = var.proxmox_host
    clone = var.template_name
    
    #now i'll set the VM specs
    agent = 1
    os_type = "cloud-init"
    cores = 2
    sockets = 1
    cpu = "host"
    memory = "2048"
    scsihw = "virtio-scsi-pci"
    bootdisk = "scsi0"
    
    disk {
      slot = 0
      size = "32G"
      type = "scsi"
      storage = "SSD"
      iothread = 1
    }
    
    #we can set the IP here, and we can use the same counter variable to set an IP.
    ipconfig0 = "ip=10.0.0.1${count.index +1}/24,gw=10.0.0.1"
    
    #include your SSH keys so you can SSH into the new server right from the start. We'll use another variable there.
    sshkeys = <<EOF
    ${var.ssh_key}
    EOF
  }
```

### Vars File
As you can see from the main.tf file above, we used a short number of variables that we'll want to pass through from our vars.tf file. You can see this here:
```
variable "ssh_key" {
	defualt = "PASTE YOUR KEY HERE"
  }
  
variable "proxmox_host" {
	default = "Zeus"
  }
  
variable "template_name" {
	defualt "ubuntu_cloud_temp"
  }
```

## Run the Plan
So now our plan is fully fleshed out and the variables are configured. Now all we have to do is run terraform plan again from the directory holding your files. This is going to detail out your Plan so that you can review it and make sure that all of the parameters are being read correctly and show up how you want them. Once you're satisfied, run terraform apply and watch the magic!
You should now be able to go to Proxmox and see the VM created and running! Should you wish to do so, you can scale your VM deployment up or down by changing the `count = 1` line in the `main.tf` file, and re-running terraform apply.

# Conclusion
If you're following along you'll notice we're somewhat starting from the ground up, and configuring an environment that eventually I can run a full Kubernetes cluster on. In a previous article, I created a VM template using Cloud-Init so that I can automate the install of VM's on Proxmox. Now we've set up Terraform to automate the deployment of those VM's, and allowed easy management of those deployments. The next logical step is to configure those VM's (updates, installing software, etc.) using a tool called Ansible. Look for my next article to see how that works!

### References
https://registry.terraform.io/providers/Telmate/proxmox/latest/docs
