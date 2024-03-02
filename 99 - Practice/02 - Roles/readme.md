# Roles

## Content

- [Create A Role](#create-a-role)
- [Define A Role](#define-a-role)
- [Use A Role](#use-a-role)

---


## Create A Role

Roles must be placed in the `roles` folder, and can be initialized manually
or using the `ansible-galaxy` init command:
```bash
ansible-galaxy init mysql_db
```
In particular can be created the following roles:
- python (with the python dependencies installation)
- mysql_db (with the database setup)
- flask_web (with the flask installation and start)


## Define A Role

After the creation, it is possible to move all the tasks under the tasks
directory to the main yaml file in the tasks folder of the associated role.

For example, the `tasks/deploy-db.yaml` can be copied into 
`roles/mysql_db/tasks/main.yaml`


## Use A Role

A role can be used in this way (order is important):

```yaml
- name: Deploy a flask web application

  hosts: node1,node2
 
  roles:
   - ping
   - python
   - mysql_db
   - flask_web

```


