# Provisioning ATTX Components on CSC Open Stack (cPouta)

This guide exemplifies how to provision 5 VMs in cPouta (CSC Open Stack) with Ansible, for forming a Docker Swarm and running the ATTX stack.

<!-- TOC START min:1 max:3 link:true update:true -->
  - [1. Preparing Ansible's control node](#1-preparing-ansibles-control-node)
  - [2. Upload and edit Ansible's configuration files](#2-upload-and-edit-ansibles-configuration-files)
  - [3. Provisioning the instances](#3-provisioning-the-instances)
  - [4. Possible problems and solutions](#4-possible-problems-and-solutions)

<!-- TOC END -->

Requirements :

1. Minimum of 3 instances created on cPouta (Ubuntu-16.04 recommended) (cf. https://research.csc.fi/pouta-flavours).

2. A Key Pair certificate must be created and associated with all instances, to allow access as "cloud-user" (cPouta default). SSH access security group must be created and associated with all instances, so that SSH communication between them is possible. Public floating IP address must be associated with one instance, for external connections (SSH, SFTP) (cf. https://research.csc.fi/pouta-getting-started).

3. The Key Pair certificate must be imported to the host computer of ATTX developer/user, in order to allow SSH and SFTP authentication to the instance associated with the the public floating IP address (cf. https://research.csc.fi/pouta-connecting-a-virtual-machine).
On linux commands to add certificates could be added by runing:
  * Start ssh agent (if not started) `eval "$(ssh-agent -s)"`
  * Add private key from a specific location: `ssh-add ~/user/cpouta.psk`

## 1. Preparing Ansible's control node

In order for your instance with the public IP address (e.g. "attx-instance-1") to be set up as Ansible's control node, you'll need to install PIP and Ansible's prerequisites:
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install python-dev
$ sudo apt-get install python-pip
```

Install Ansible:
```
$ sudo apt-get install ansible
```


## 2. Upload and edit Ansible's configuration files

Via SFTP or SCP, transfer the contents of  <github_repo_link> to your instance in cPouta with the floating public IP address (e.g. "attx-stack-1"), namely to the instance that you have set up as the Ansible control node for this exercise.

The list of the files is:

* `inventory` (hostname and IP address information used by Ansible for setting up the instances with Docker and Docker-Compose, **must be edited** as per your instances hostnames and IP addresses)

* `cpouta.yml` (hostname and IP address information used by Ansible for building up /etc/hosts in all instances, **must be edited** as per your instances hostnames and IP addresses)

* `provisioning.yml` and `ansible.cfg` (Ansible's provisioning and configuration files, **don't edit**)

* `ssh-config` and `private-key` (SSH configuration and private key for Ansible, **don't edit**)

The recommended path for the Ansible files is `/home/cloud-user/provision-swarm` (**must be created**).


## 3. Provisioning the instances

From Ansible's control node, run as cloud-user:
```
$ ansible-playbook provisioning.yml
```

You can display extra verbose information for troubleshooting with the `-vvvv` option, e.g. "`ansible-playbook -i hosts provisioning.yml -vvvv`"

## 4. Possible problems and solutions

- If you get strange Ansible errors about dependencies, try to check your pip
  version with `pip --version`. The current version is 9.0.1. If your pip is
  older than this, upgrade it with `sudo pip install --upgrade pip`, restart
  your terminal session and install the Ansible prerequisites again.

- If Ansible fails to pip-install docker-compose on a node complaining about a pip module's version, try to to    upgrade that module with `sudo pip install -upgrade <module>` (e.g. `sudo pip install --upgrade six`) , and run the Ansible provisioning again. You can find in Github repo <link_here> a list of pip modules known to work ("pip.requirements.txt"), which can be installed with `sudo pip install -r pip.requirements.txt`.

- If you get a ssh error saying `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED`
  it means that you already had in the past some other host using one of the
  IP addresses we use here as example. To solve this, remove the old entry in your SSH
  `known_hosts` file with:
```
  $ ssh-keygen -f "~/.ssh/known_hosts" -R <INSTANCE_IP_ADDRESS> -R <INSTANCE_IP_ADDRESS> -R <INSTANCE_IP_ADDRESS> -R <INSTANCE_IP_ADDRESS> -R <INSTANCE_IP_ADDRESS>
```