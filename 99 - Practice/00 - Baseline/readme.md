# Baseline

## Content

- [Manual Setup](#manual-setup)
- [Playbook](#playbook)

---


## Manual Setup

It is possible the setup the environment manually by logging in the container
using ssh:
```bash
ssh root@127.0.0.1 -p 9001
```
and by running the following commands manually:
```bash
apt-get install -y python3 python3-setuptools python3-dev build-essential python3-pip python3-mysqldb

pip3 install Werkzeug==2.2.2 Flask==2.1.3 Flask-MySQL==1.5.2 cryptography==42.0.5

apt install -y mysql-server mysql-client
service mysql start

echo "CREATE DATABASE employee_db;
CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'Passw0rd';
GRANT ALL ON employee_db.* TO 'db_user'@'localhost';
USE employee_db;
CREATE TABLE employees (name VARCHAR(20));
INSERT INTO employees VALUES ('John Doe'); " > init.sql

mysql -u root < init.sql

echo "import os
from flask import Flask
from flaskext.mysql import MySQL      # For newer versions of flask-mysql
# from flask.ext.mysql import MySQL   # For older versions of flask-mysql
app = Flask(__name__)

mysql = MySQL()

mysql_database_host = 'MYSQL_DATABASE_HOST' in os.environ and os.environ['MYSQL_DATABASE_HOST'] or  'localhost'

# MySQL configurations
app.config['MYSQL_DATABASE_USER'] = 'db_user'
app.config['MYSQL_DATABASE_PASSWORD'] = 'Passw0rd'
app.config['MYSQL_DATABASE_DB'] = 'employee_db'
app.config['MYSQL_DATABASE_HOST'] = mysql_database_host
mysql.init_app(app)

conn = mysql.connect()

cursor = conn.cursor()

@app.route('/')
def main():
    return 'Welcome!'

@app.route('/how-are-you')
def hello():
    return 'I am good, how about you?'

@app.route('/read-from-database')
def read():
    cursor.execute('SELECT * FROM employees')
    row = cursor.fetchone()
    result = []
    while row is not None:
      result.append(row[0])
      row = cursor.fetchone()

    return ','.join(result)

if __name__ == '__main__':
    app.run()
" > app.py

FLASK_APP=app.py flask run --host=0.0.0.0
```

After that it is possible to connect from the host to the web server
using the following urls:
- http://localhost:5001/
- http://localhost:5001/how-are-you
- http://localhost:5001/read-from-database


## Playbook

Now we can go step by step to create the same setup using Ansible

### inventory
First of all it is necessary to define the inventory file with the information
of the nodes
```
node1 ansible_host=127.0.0.1 ansible_port=9001 ansible_user=root ansible_pass=node1-secret
node2 ansible_host=127.0.0.1 ansible_port=9002 ansible_user=root ansible_pass=node2-secret
```

### Ping
After that it is possible to test the connection defining a playbook with the simple ping
```yaml
- name: Deploy a flask web application
  hosts: node2 # just on the not configured node
  tasks:
   - name: Ping
     ping:
```

### Install dependencies

It is possible to install the python and MySQL dependencies by using the apt
module and require the state present:
```yaml
   - name: Install python dependencies
     apt: name='{{ item }}' state=present
     with_items:
      - python3 
      - python3-setuptools 
      - python3-dev
      - build-essential 
      - python3-pip
      - python3-mysqldb

   - name: Install database dependencies
     apt: name='{{ item }}' state=present
     with_items:
      - mysql-server
      - mysql-client
```

### Setup MySQL

In order to setup MySQL, it is first necessary to start the server:
```yaml
   - name: Start mysql service
     # this is not working due to the docker environment
     # service:
     #   name: mysql
     #  state: started
     #  enabled: yes
     # it is necessary to use the command explicitly
     command: service mysql start
```
it is then possible to setup the database using the MySQL specific modules:
```yaml
   - name: Create mysql db
     mysql_db: 
       name: employee_db
       state: present
       login_unix_socket: /run/mysqld/mysqld.sock

   - name: Initialize database
     mysql_query:
       login_db: employee_db
       query:
        # workaround to ensure that the creation is idempotent
        - CREATE TABLE IF NOT EXISTS employees (name VARCHAR(20) UNIQUE);
        - INSERT IGNORE INTO employees VALUES ('John Doe');
       login_unix_socket: /run/mysqld/mysqld.sock

   - name: Create mysql user
     mysql_user: 
       name: db_user
       password: Passw0rd
       priv: '*.*:ALL'
       host: '%'
       state: present
       login_unix_socket: /run/mysqld/mysqld.sock
```

### Setup python

It is possible to install the needed packages using the pip module:
```yaml
   - name: Install python dependencies
     pip: 
      name: "{{ item }}"
      state: present
     with_items:
      - Werkzeug==2.2.2
      - Flask==2.1.3
      - Flask-MySQL==1.5.2
      - cryptography==42.0.5
```
and after that copy the application code:
```yaml
   - name: Copy source code
     copy: 
      src: app.py
      dest: /opt/app.py
```

### Start the application

Finally it is possible to start the application using the shell command:
```yaml
   - name: Start the web service
     shell: FLASK_APP=/opt/app.py flask run --host=0.0.0.0
```

### Run the playbook

The playbook can be run using:
```bash
ansible-playbook playbook.yaml -i inventory 
```

After the run is finished, it is possible finally to reach the new server:
- http://localhost:5002/
- http://localhost:5002/how-are-you
- http://localhost:5002/read-from-database


