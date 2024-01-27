# Intro

## Content

- [What is Ansible ](#what-is-ansible)
- [Installation](#installation)
- [Configuration](#configuration)
- [Inventory](#inventory)

---

## What is Ansible

Ansible is used to automate complex deployments.\
It can replace a lot of bash script lines in a few yaml ones.

## Installation

For the installation of
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
is used 
[pipx](https://pipx.pypa.io/stable/)

```bash
# install pipx
sudo apt install pipx
# ensure pipx is correctly installed
pipx ensurepath

# install ansible
pipx install --include-deps ansible
# verigfy ansible is correctly installed
ansible --version

# upgrade ansible
pipx upgrade --include-injected ansible
```

## Configuration

After the installation, Ansible should crate a configuration file at
`/etc/ansible/ansible.cfg` in ini format, it is the location of the default
configurations.

Is is also possible to create a fully documented configuration file in this way:
```bash
# basic
ansible-config init --disabled > ansible.cfg
# include all plugins
ansible-config init --disabled -t all > ansible.cfg
```


The configuration file in the default location will contain the default values,
other than that, a configuration file can be set up in any working directory
and it will override the defined variables.

The configuration file's path can also be defined in the `ANSIBLE_CONFIG`
environment variable.

The precedence of the variable specification is:
- environment variable file
- current working directory file
- user home directory file
- default location file

Further is also possible to specify single properties as environment variables,
usually the naming convention is `ANSIBLE_<property>` but it is always better
to check the 
[documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
first.

In order to list all the available configurations it is possible to use:
```bash
ansible-config list
```
To see the current active configuration file can be used:
```bash
ansible-config view
```
To see the current configuration can be used:
```bash
ansible-config dump
```

## Inventory

Ansible uses ssh or powershell to configure one or multiple system in the
infrastructure at the same time.

This allow Ansible to work without installing additional tools on the target
machine.

The target host systems are stored in an 
[inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)
file, that is located in `/etc/ansible/hosts`.

The target hosts can be configured in this way:
```ini
server1.example.com
server2.example.com

[group1_name]
server3.example.com
server4.example.com

[group2_name]
server5.example.com
server6.example.com
```

### Alias

It is possible to define aliases for the different hosts in this way:
```ini
web1 ansible_host=server1.example.com
web2 ansible_host=server2.example.com
```

In the case the  `ansible_host` must be explicitly set.

### Configuration

The server can be configured with additional parameter:
- `ansible_host`: the hostname of the server
- `ansible_connection`: the type of connection `ssh`, `winrm` or `localhost`
- `ansible_port`: the port user for the connection, default 22
- `ansible_user`: the user name to log, default root for Linux and administrator
for Windows.
- `ansible_ssh_pass`: the password for ssh
- `ansible_password`: the password for winrm

The different properties are set on the same line with a space separator:
```ini
web ansible_host=server1.example.com ansible_connection=ssh

localhost ansible_connection=localhost
```

### Advanced Groups

The different targets, after the assign of the alias, can be added to multiple
groups, and the groups can also be grouped:
```ini
# definition of hosts
web1 ansible_host=server1.example.com
web2 ansible_host=server2.example.com

db1 ansible_host=database1.example.com
db2 ansible_host=database2.example.com

# parent
[all_servers:children]
web_servers
db_servers

[web_servers]
web1
web2

[db_servers]
db1
db2

[first_region]
web1
db1

[second_region]
web2
db2
```

Using grouping it is possible to define common configuration in the parent
and add specific configurations on each child.

### Yaml

For complex organizations the `ini` format showed above is not easy to manage.

In this case it is possible to use the yaml for a cleaner and more flexible
definition.

```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
```

