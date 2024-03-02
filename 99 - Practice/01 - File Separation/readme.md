# File Separation

## Content

- [Vars](#vars)
- [Vars File](#vars-file)
- [Tasks File](#tasks-file)

---


## Vars

In the vars section can be defined the different variables of the playbook:
```yaml
  vars:
    db_name: employee_db
    db_user: db_user
    db_pass: Passw0rd
    db_table: employees
    employee_name: "John Doe"
```
and use them in this way:
```yaml
   - name: Create mysql db
     mysql_db: 
       name: "{{ db_name }}"
       state: present
       login_unix_socket: /run/mysqld/mysqld.sock

   - name: Initialize database
     mysql_query:
       login_db: "{{ db_name }}"
       query:
        - "CREATE TABLE IF NOT EXISTS {{ db_table }} (name VARCHAR(20) UNIQUE);"
        - "INSERT IGNORE INTO employees VALUES ('{{ employee_name }}');"
       login_unix_socket: /run/mysqld/mysqld.sock

   - name: Create mysql user
     mysql_user: 
       name: "{{ db_user }}"
       password: "{{ db_pass }}"
       priv: '*.*:ALL'
       host: '%'
       state: present
       login_unix_socket: /run/mysqld/mysqld.sock
```


## Vars File

To be more clean, it is better to move them in a separate file, there are two
possible approaches:
- if each host needs a separate configuration:\
in this case it is possible to create for each host an 
`host_vars/<host_name>.yaml` file.
- if all the hosts can have the same configuration:\
in this case, in order to avoid duplication, it is better to put the hosts
under the same group and create a `group_vars/<group_name>.yaml` file.

Combining the two approaches, it is possible to move the hosts info to the
`host_vars` and the playbook common variables to the `group_vars`.

The inventory will than remove all the variables and define a group:
```
[nodes]
node1
node2
```
the hosts info will be moved to the `host_vars/nodes.yaml`:
```yaml
# host_vars/node1.yaml
ansible_host: 127.0.0.1
ansible_port: 9001
ansible_user: root
ansible_pass: node1-secret
```
```yaml
# host_vars/node2.yaml
ansible_host: 127.0.0.1
ansible_port: 9002
ansible_user: root
ansible_pass: node2-secret
```
and in the `group_vars/nodes.yaml`:
```yaml
db_name: employee_db
db_user: db_user
db_pass: Passw0rd
db_table: employees
employee_name: "John Doe"
```

## Tasks File

Further also the tasks can be moved to dedicated yaml files, these files must
be placed in the `tasks` folder.

The new files can be defined in this way:
```yaml
# deploy-db.yaml
- name: Install database dependencies
  apt: name='{{ item }}' state=present
  with_items:
   - mysql-server
   - mysql-client

- name: Start mysql service
  command: service mysql start

- name: Create mysql db
  mysql_db: 
    name: "{{ db_name }}"
    state: present
    login_unix_socket: /run/mysqld/mysqld.sock

- name: Initialize database
  mysql_query:
    login_db: "{{ db_name }}"
    query:
     - "CREATE TABLE IF NOT EXISTS {{ db_table }} (name VARCHAR(20) UNIQUE);"
     - "INSERT IGNORE INTO employees VALUES ('{{ employee_name }}');"
    login_unix_socket: /run/mysqld/mysqld.sock

- name: Create mysql user
  mysql_user: 
    name: "{{ db_user }}"
    password: "{{ db_pass }}"
    priv: '*.*:ALL'
    host: '%'
    state: present
    login_unix_socket: /run/mysqld/mysqld.sock
```
```yaml
# deploy-flask.yaml
- name: Install python dependencies
  pip: 
    name: "{{ item }}"
    state: present
  with_items:
   - Werkzeug==2.2.2
   - Flask==2.1.3
   - Flask-MySQL==1.5.2
   - cryptography==42.0.5

- name: Copy source code
  copy: 
    src: app.py
    dest: /opt/app.py

- name: Start the web service
  shell: FLASK_APP=/opt/app.py nohup flask run --host=0.0.0.0 &
```
and included in the playbook using the include_tasks module:
```yaml
   - include_tasks: tasks/deploy-db.yaml

   - include_tasks: tasks/deploy-flask.yaml

```


