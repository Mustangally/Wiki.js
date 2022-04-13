---
title: Learning the Basics of Ansible
description: 
published: true
date: 2022-04-13T00:23:47.001Z
tags: linux, ansible, automation
editor: markdown
dateCreated: 2022-03-23T23:58:27.650Z
---

# What is Ansible?
Per the Ansible documentation, "Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows."

Ok but what does that mean? It means that rather than individually and manually configuring and utilizing servers, we can build code files that define how and what the servers should be running. We can address multiple servers at once so that we aren't configuring 1 at a time, and through version control we can see where we were and how we got to where we are now. This allows us to have a better view on what is going on in our environment.

Today we will go over a very basic use case for Ansible so that we can get a feel for what its capable of and how it works. Ansible is incredibly powerful and can handle far more than we'll do today, but we need to walk before we can run.

# Project Outline

What we're going to explore today is configuring a set of servers (6 of them to be exact) by updating them, installing a qemu-guest-agent so our host Hypervisor can manage them, and installing docker to lay the foundation for Kubernetes. The end goal is to build a full functioning Kubernetes cluster, but automate the process with Ansible so that we don't have to manually configure and join each node.

If you've seen my previous articles, we started from scratch with creating VM Templates on Proxmox and then deploying those servers using Terraform. The next logical step then is setting up those linux servers to run our workload.

# Installing Ansible

My "dev" machine is running Ubuntu, so installing ansible is rather straightforward. There is already an apt package built for this, so all we have to do is:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ansible
sudo apt install sshpass
```

> **NOTE:** The sshpass utility is designed to run SSH using the keyboard-interactive password authentication mode, but in a non-interactive way. SSH uses direct TTY access to ensure that the password is indeed issued by an interactive keyboard user.
> 
{.is-info}

# Setting the Stage

Ok we have ansible installed on our Dev machine. Now what? Well we need to start building our config files. We're going to use 2 primary file types today; `hosts`, and `playbooks`.

Lets create a project directory:
```
mkdir learning-ansible
cd learning-ansible
```

## Host File

Our Host file is going to define the "Hosts" or servers that we want to address. You can group hosts within the file so that you can address the group for different tasks, but we're only going to make 1 group today. You can simply extend the example though if you want to define more groups.

Now we'll create two directories within our project directory:
```
mkdir inventory
mkdir playbooks
```
We're going to work with the inventory directory for now. We'll make a file called `hosts`
```
[ubuntu]
10.0.0.110
10.0.0.111
10.0.0.112
10.0.0.150
10.0.0.151
10.0.0.152
```
> Notice the `"[ubuntu]"` line. This is our group. You can make more of these within the hosts file, and we'll see how to address the group shortly.
{.is-info}

Now that you have your hosts defined, we can test our host files by running this command from our project directory:
```
ansible -i ./inventory/hosts ubuntu -m ping --user kevin --ask-pass
```

As you might be able to see, we selected the "ubuntu" group by directing ansible to our hosts file, then specifying the group, then the command we want to run followed by the user to run it as.

The output should look something like this (one entry for each host):
```
10.0.0.112 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Playbooks

Now that we have our hosts defined, we can look at the second type of file we need to work with today, and that's going to be a `"playbook"`

A playbook is kind of what it sounds like. It defines what we want ansible to actually do. There are almost endless options depending on how creative you are, but we'll go over some pretty basic ones just to see what they look like.

We can create an `inventory` directory in our project parent directory to hold our playbooks.

First we can make one to update all of our hosts by running an `apt update` command on each.
`apt-update.yaml`
```
- hosts: "*"
  become: yes
  tasks:
    - name: apt
      apt:
        update_cache: yes
        upgrade: 'yes'
```
> Notice that these are in yaml formatting.
{.is-info}

To run the playbook, we can run this command:
```
ansible-playbook ./playbooks/apt.yml --user kevin --ask-pass --ask-become-pass -i ./inventory/hosts ubuntu
```
Lets break down this command a little bit. Like before we're first directing ansible to the playbooks directory and selecting the playbook we want to run. Then we're telling ansible which user to use for authentication to the host. We're prompting for the user's ssh password, and then asking for the sudo password and running the command as sudo. Then we point Ansible to the inventory host file we want to use, and picking a group within that.

The expected result is something like:
![screenshot_from_2022-03-23_17-54-25.png](/screenshot_from_2022-03-23_17-54-25.png)

You'll notice that Ansible confirms that it can connect to the hosts, then it runs the task you created, and give you a summary at the end to see what was done.

# Other Considerations

### Host Key Checking
Ansible has host key checking enabled by default.

If a host is reinstalled and has a different key in `known_hosts`, this will result in an error message until corrected. If a host is not initially in `known_hosts` this will result in prompting for confirmation of the key, which results in an interactive experience if using Ansible, from say, cron. You might not want this.

If you understand the implications and wish to disable this behavior, you can do so by editing `/etc/ansible/ansible.cfg` or `~/.ansible.cfg`:
```
[defaults]
host_key_checking = False
```
Alternatively this can be set by the `ANSIBLE_HOST_KEY_CHECKING` environment variable:

`$ export ANSIBLE_HOST_KEY_CHECKING=False`

# Conclusion

That's really all we're going to discuss in this article. In another article we will begin configuring our hosts with the software we need and then joining them to our K3S cluster.

---

You can see my GitHub for more examples of playbooks as I come up with more.
https://github.com/Mustangally/Ansible