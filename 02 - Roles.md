# Roles

## Content

- [Roles](#roles)
- [Ansible Galaxy](#ansible-galaxy)
- [Collections](#collections)

---

## Roles

It is possible to assign roles to the different hosts like mysql, nginx,
backup, etc.

This mean that will be performed all the operations needed to transform the
service to represent that specific role, like installing all the pre-required
software and configuring it.

The main idea is that the common operations are always the same, so they can
be placed inside a role and reused or shared with others.

```yaml
- name: Play Role
  hosts: db-servers
  roles:
    - mysql
```

Further roles introduce best practices, like:
- put vars in the var directory
- put tasks in the tasks directory
- put defaults in the defaults directory
- put handlers in the handlers directory
- put templates in the templates directory
- document in meta directory and in the base readme

## Ansible Galaxy

A place where can be found a lot of shared roles is
[Ansible Galaxy](https://galaxy.ansible.com/ui/).

The `ansible-galaxy` command line tool allows to create new roels or use 
existing ones.

### Create New Roles

It is possible to create a new role in this way:
```bash
ansible-galaxy init <role_name>
```
The following directory structure will be created:
```
.
└── role_name
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

In order to later use this role in a `my_playbook`, the role folder must be
moved in two different places:
- in the `my_playbook/roles` folder:\
so that it will be available only to that playbook.
- in the shared `/etc/ansible/roles`:\
so that it will be available to any playbook.

### Use Existing Roles

rExisting roles can be searched in the ui or in the command line in this way:
```bash
ansible-galaxy search <role_name>
```
after that it is possible to install it using
```bash
ansible-galaxy install <role_name>
```

### Privileges

It is possible to escalate the privileges using the `become` directive:
```yaml
- name: Play Role
  hosts: db-servers
  roles:
    - role: mysql
      become: true
```
or pass variable:

```yaml
- name: Play Role
  hosts: db-servers
  roles:
    - role: mysql
      vars:
        mysql_user_name: db-user
```

## Collections

Collections are a way to package and distribute modules, roles, plugins etc.
It is a simple unit that contains multiple objects at once and cam be created
by a vendor or by the community.

The benefits are:
- expanded functionality
- modularity and re-usability
- simplified distribution and management

A collection can be found in the Ansible Galaxy and installed using the
command line tool:
```bash
ansible-galaxy collection install <collection_name>
```

It is possible to define a list of collections to use in this way:
```yaml
collections:
  - name: collection.one
    version: '1.0.0'
  - name: collection.two
    version: '2.0.0'
```
and install all at once using
```bash
ansible-galaxy collection install -r requirements.yml 
```

