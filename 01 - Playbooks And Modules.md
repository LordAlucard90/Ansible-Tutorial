# Playbooks And Modules

## Content

- [Playbook](#Playbook)
- [Ansible-Lint](#ansible-lint)
- [Variables](#variables)
- [Magic Variables](#magic-variables)
- [Facts](#facts)
- [Loops](#loops)
- [Conditionals](#conditionals)
- [Modules](#modules)
- [Plugins](#plugins)
- [](#)
- [](#)

---

## Playbook

An Ansible Playbook consists in an yaml file where are defined a series of
tasks that that can be easy or very complex.

A play in a playbook defines a set of activities to be run on a single or a 
group of hosts.

A task in a play is a single action to be performed on an host, like:
- execute a command
- run a script
- install a package
- shutdown or restart

While a play is structured as a dictionary, where the variables can be placed
in any order, the tasks are a list, therefore the order is important.


An example of playbook is:
```yaml
- name: Play 1
  hosts: localhost
  tasks:
    - name: Print date
      command: date

    - name: Say hello
      command: 'echo "Hello"'


- name: Play 2
  hosts: localhost
  tasks:
    - name: Say hello
      command: 'echo "Hello"'

    - name: Print date
      command: date
```

### Play Parameters

The `hosts` indicates on which host or group this play should be run.

Each host specified in the playbook must match the list of hosts in the
inventory file, all the connection information are retrieved from the inventory.

The consideration are valid when in the hosts is specified a group, in that 
case the play is performed on all the hosts that belong to that group 
simultaneously.

### Modules

The different actions run by a task are called modules, some of them are:
- command
- script
- yum
- service

See below for more details.

### Run And Verify Playbooks

It is possible to run a Playbook using
```bash
ansible-playbook <playbook_name>.yaml
```

It is possible to verify a Playbook before running it using two node:
- Check Mode
- Diff Mode
- Syntax check Mode

### Check Mode

In the check mode Ansible will execute the playbook without actually making any
kind of change in the hosts, therefore it is possible to see all the changes
would be done, without actually apply them.

This mode can be enabled using the `--check` option.
```bash
ansible-playbook <playbook_name>.yaml --check
```

All the modules that do not support this mode will be skipped if this mode 
is enabled.

### Diff Mode

The diff mode shows the differences between the current status and the new 
status after the apply of the changes but without actually making them.
It provides a before and after report.

This mode can be enabled using the:
```bash
ansible-playbook <playbook_name>.yaml --check --diff
```

### Syntax check Mode

This mode provides a quick check to ensure the playbook will not fail for
syntax errors.

This mode can be enabled using the `--syntax-check` options.

```bash
ansible-playbook --syntax-check <playbook_name>.yaml
```

## Ansible-Lint

[Ansible-Lint](https://ansible.readthedocs.io/projects/lint/)
is a command line tool that performs linting on Ansible playbooks, roles 
and collections.

It check the code for potential errors, bugs, stylistic errors and suspicious
constructs.

It can be installed using pip:
```bash
pip3 install ansible-lint
```
and used on a playbook in this way:
```bash
ansible-lint <playbook_name>.yaml
```

## Variables

Variables can be defined in a dedicated files named following the tag name,
for example `<tag_name>.yaml`.

After that can be referenced using Jinja2 Templating: `'{{ variable_name }}'`,
it is important to enclose it inside single quotes `'...'`.

### Variable Types

The supported types are:
- String, cam be without quotes, with single `'` or double '"'
- Number (integer or floating point)
- Boolean
- List, for a ordered collection of values
- Dictionary, for collections of key-value pairs, both can be of any type.

A specific element of a list can be accessed using `{{ elements[<n>] }}`,
with n starting from 0, to the number of the elements minus one.

A specific element of a dictionary can be accessed using
`{{ dictionary.<key> }}` or `{{ dictionary.['<key>'] }}` .

Arrays of dictionaries can be represented in this way
```yaml
examples:
  - letter: a
    position: 0
  - letter: b
    position: 1
  - letter: c
    position: 2
```
or using the json format:
```yaml
examples:
  - { letter: a, position: 0 }
  - { letter: b, position: 1 }
  - { letter: c, position: 2 }
```

### Variable Precedence

Levels:
```ini
# host level
web1 ansible_host=server1.example.com
web2 ansible_host=server2.example.com dns_server=4.4.4.4

[web_servers]
web1
web2

[web_servers:vars]
# group level
dns_server:8.8.8.8
```

When Ansible is run, first associate the groups variables and then the host 
variables that will override the first ones.

After that the variable defined in the playbook are the latest to be computed,
therefore will override the group and host ones.

Last if a variable is defined in the command line will override all the others.

The full list of variables with their precedence can be found 
[here](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)

### Commands Output

In order to print the output of a command, it is possible to use use the
`register` directive and use the same variable in the `debug`:
```yaml
- name: Check hosts file
  hosts: all
  tasks:
    - shell: cat /etc/hosts
      # the output is stored in a variable name result
      register: result

    - debug:
        # the debug print the content of the data inside result
        var: result
```

Each module outputs data in a different format, that is usually a dictionary.\
It is possible to print a specific part, like the return code referencing
the key of the dictionary like `result.rc` to the return code.

Each variable created with `register` directive follows the scope of the host
and is available in the playbook execution.

Another way to use the debug module, is to add the `-v` flag:
```bash
ansible-playbook -i inventory playbook.yml -v
```

### Scoping

The scope defines the visibility of a variable, the different scopes are:
- Host Scope
- Playbook Scope
- Global Scope

#### Host Scope

A variable defined for an host is only visible in a play by that host.

An host variable can be defined directly in the host, in a group or in a parent
group.\
During a run the variable will be associated to the host, so there is just one 
scope, no matter where it is defined.

#### Playbook Scope

In this case a variable is only defined a play, and will be not visible in the
others:
```yaml
- name: Play1
  hosts: web1
  vars:
    example: "Example"

  tasks:
    - debug:
        var: example

- name: Play2
  hosts: web2
  tasks:
    - debug:
        var: example
```
In this case the second run will print an empty row.

#### Global Scope

A variable in this case is accessible in all the places, this type of variables
can be defined in the command line using `--extra-vars "<key>=<value>"`.

## Magic Variables

It is possible to access Informations outside the scope using magic variables,
these magic variables are:
- `hostvars`
- `groups`
- `group_names`
- `inventory_hostname`

The complete list of magic variables can be found in the
[special variables documentation](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html).


#### hostvars

It allows to access the variables defined in other hosts:
```
{{ hostvars['<host_name>'].<key> }}
```

More informations about the host can be retrieved using `ansible_facts`:
```
{{ hostvars['<host_name>'].ansible_facts.<key> }}
```
Some of the properties are:
- architecture
- devices
- mounts
- processor

#### groups

It returns all hosts under a give group:
```
{{ groups['<group_name>'] }}
```

#### group_names

It returns all groups in which is an hosts:
```
{{ group_names }}
```

#### inventory_hostname

Returns the name configured for the host in the inventory file:
```
{{ inventory_hostname }}
```

## Facts

When Ansible connects to a target machine, it will collect different information
of that machine known as facts.
Some of this data is:
- system architecture
- os version
- processor details
- memory details
- network connectivity
- ip addres
- disks
- volumes
- mounts
- date and time 

This process is performed by the setup module.

All this info are available in the `ansible_facts` variable.

To disable them can be used `gather_facts` in the yaml:
```yaml
- name: Play Example
  hosts: all
  gather_facts: no
```
or `gathering` in the ini:
```ini
# default implicit (yes)
gathering = explicit 
```

## Loops

Using the `loop` directive it is possible to repeat the same task multiple 
times with different arguments.\
The current value is stored in an `item` variable.

An example of loop to add a list of users is:
```yaml
- name: Play Loop
  hosts: localhost
  tasks:
    - name: "Add the user: {{ item }}"
      user: name='{{ item }}' state=present
      loop:
        - first
        - second
        - third
```

The loop also supports arrays of dictionaries:
```yaml
- name: Play Loop
  hosts: localhost
  tasks:
    - name: "Add the user: {{ item.name }}"
      user: name='{{ item.name }}' state=present uid='{{ iten.uid }}'
      loop:
        - name: first
          uid: 2001
        - sname: econd
          uid: 2002
        - tname: hird
          uid: 2003
```

The `loop` directive is actually the new way of defining loops, before was used
the `with_items` one.

It is also possible to iterate through other items like:
- `with_file`: files
- `with_url`: urls
- `with_mongodb`: mongo databases

It is possible to find other examples in the
[loop documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html)


## Conditionals

It is possible to use a `when` statement to perform or not a specific task.

For example can be choose the package managed in this way:
```yaml
- name: Play Conditional
  hosts: all
  tasks:
    - name: Install NGINX on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install NGINX on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"
```

Check:
- disjunction: `or`
- conjunction: `and`
- comparison: `<`, `<=`, `==`, `>`, `>=`


In a loop they can used in this way:
```yaml
- name: Play Conditional
  hosts: all
  vars:
    packages:
      - name: nginx
        required: true
      - name: mysql
        required: true
      - name: apache
        required: false
  tasks:
    - name: "Install {{ item.name }}"
      apt:
        name: "{{ item.name }}"
        state: present
      when: item.required == true
      loop: "{{ packages }}"
```

Or it can be used to send a conditional email if a service is down:
```yaml
- name: Play Conditional
  hosts: localhost
  tasks:
    - name: Check httpd status
      command: service httpd status
      register: result

    - name: Notify with email
      email:
        to: to@example.com
        subject: Service is down
        body: The httpd service is down
      when: result.stdout.find('down') != -1
```

The facts are used in conditionals to skip or perform specific tasks, as:
```yaml
- name: Play Conditional
  hosts: localhost
  tasks:
    - name: Install Ngnix on Ubuntu 18.04
      apt:
        name: nginx=1.18.0
        state: present
      when: ansible_facts['os_family'] == 'Debian' and ansible_facts['distribution_major_version'] == 18
```

## Modules

Modules in Ansible can be categorized based on the functionality:
- System:\
user, group, hostname, mount, timezone, sistemd, service, etc.
- Commands:\
command, expect, raw, script, shell, etc.
- Files:\
archive, copy, file, find, lineinfile, resplace, stat, unarchive, etc.
- Database:\
mongodb, mysql, posgresql, etc.
- Cloud:\
amazon, azure, digital ocean, docker, google, vmware, etc.
- and a lot more.

The complete list can be found in the
[modules documentation](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)
or in the terminal using
```bash
ansible-doc -l
```

A module can take the execution info in two ways:
- parametrized input:\
the parameters are well defined, for example `copy: src=my_src dest=my_dest`.
- free form:\
there is not a standard way to define the input, for example `command: date` or 
`command: mkdir /my_dir creates=/my_dir` or `command: cat my.conf chdir=/etc`.

Some useful modules are:
- command:\
executes a command.
- script:\
runs a script after transferring it.
- service:\
maintains services, as start, stop, restart (use started not start).
- lineinfile:\
searches a line and replaces or adds it.
- shell:\
similar to command but executes the command in a shell having access
to environment variables and redirection (`>>`, `<<`).

```yaml
- name: Play Conditional
  hosts: localhost
  tasks:
    ## command ##
    - command: date
    - command: mydir my_dir creates=my_dir
    - command: cat my.conf chdir=/etc
    # is the same as
    - command: cat /etc/my.conf

    ## copy ##
    - copy: src=my_src dest=my_dest

    ## script ##
    - script: /path/to/script.sh -arg1 -arg2

    ## service ##
    - service: name=postgresql state=started
    # is the same as
    - service:
        name: postgresql
        state: started

    ## lineinfile ##
    - lineinfile:
        path: /path/to/file
        line: 'just some content'
```

### Idempotency

When performing actions it is important to try to have Idempotency,
this allows the script to run N times having always the same result.

An example of Idempotency is the service, in this case it is a good practice
to set the requested state as `started`, therefore if the service is not running
it is started, but if it is already running nothing will be done.

## Plugins

A plugin modifies or extends the functionality of Ansible, they can be used
on inventory, module, action, callback and more.

More in detail:
- inventory plugins:\
allow to fetch real time information on the cloud resources.
- module plugins:\
allow to provision cloud resources with custom configurations.
- action plugins:\
allow to simplify the load balancing, certificates and health checks.
- lookup plugins:\
allow to fetch data from databases or APIs.
- filter plugins:\
add additional data manipulation and transformation.
- connection plugins:\
allow to connect and communicate with various systems as ssh or docker.
- callback plugins:\
allow to define life-cycle hooks.

It is possible to search for specific elements in the
[modules and plugins index](https://docs.ansible.com/ansible/latest/collections/all_plugins.html#all-modules-and-plugins).


