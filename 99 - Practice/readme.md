# Practice

## Content

- [Intro](#intro)
- [Setup](#setup)
- [Run](#run)
- [Reset](#reset)
- [Additional Operations](#additional-operations)

---

## Intro

This section contains some practical exercises, basically it starts
from the deploy of a simple application and improves the structure
on each step.

In order to simulate two hosts I'm using docker.

Since I had troubles connecting using just the password form Ansible, I needed
to configure ssh to use the SSH key.

The different sections are:
1. [Baseline](./00%20-%20Baseline/readme.md)
    - [Manual Setup](./00%20-%20Baseline/readme.md#manual-setup)
    - [Playbook](./00%20-%20Baseline/readme.md#playbook)
1. [File Separation](./01%20-%20File%20Separation/readme.md)
    - [Vars](./01%20-%20File%20Separation/readme.md#vars)
    - [Vars File](./01%20-%20File%20Separation/readme.md#vars-file)
    - [Tasks File](./01%20-%20File%20Separation/readme.md#tasks-file)
1. [Roles](./02%20-%20Roles/readme.md)
    - [Create A Role](./02%20-%20Roles/readme.md#create-a-role)
    - [Define A Role](./02%20-%20Roles/readme.md#define-a-role)
    - [Use A Role](./02%20-%20Roles/readme.md#use-a-role)


## Setup

The steps necessary to correctly configure the local environment are:
```bash
# 1. Generate an ssh key
ssh-keygen -t rsa -b 4096

# 2. Start the containers
docker compose up -d

# 3. Verify the containers are up and running
docker ps

# 4. Copy the host key to the containers
#    (the passwords are defined in the docker-compose file)
ssh-copy-id -p 9001 root@127.0.0.1
ssh-copy-id -p 9002 root@127.0.0.1

# 5. Verify you can connect using ssh and store certificate
ssh -p 9001 root@127.0.0.1
ssh -p 9002 root@127.0.0.1
# Alternatively use
# ssh-keyscan -p 9001 -H 127.0.0.1 >> ~/.ssh/known_hosts
# ssh-keyscan -p 9002 -H 127.0.0.1 >> ~/.sshknown_hosts

# 6. Verify Ansible can correctly log into the containers
ansible all -m ping -i inventory
```


## Run

The playbook can be run from the folder by using
```bash
ansible-playbook playbook.yaml -i inventory
```


## Reset

Between each exercise it is necessary to reset the containers, it is possible
to do so by running:
```bash
# 1. Delete the containers, but keep the volumes with ssh data
docker compose down

# 2. Recreate the containers
docker compose up -d
```


## Additional Operations

If something is not working these commands can result useful:
```bash
# To delete a wrong ssh key (when delete also the volume)
ssh-keygen -f "~/.ssh/known_hosts" -R "[127.0.0.1]:9001"
ssh-keygen -f "~/.ssh/known_hosts" -R "[127.0.0.1]:9002"

# Increase the ansible debug level
ansible node1 -m ping -i inventory.txt -vvv


# Check the container's ip
docker inspect node1 | jq '.[0].NetworkSettings.Networks.docker_default.IPAddress'

# log using the container's ip
ssh root@172.18.0.2
```


