# Security

- [Vault](#vault)
- [Dynamic Inventory](#dynamic-inventory)

---

## Vault

It is possible to store password in a more secure way as plain text, using vault.

To use it, it is possible to run the `ansible-vault` encrypt command 
(a password is required):
```bash
ansible-vault encrypt <inventory_file>
```
It is then possible to use the encrypted inventory file in this way:
```bash
# manually insert the password
ansible-playbook <playbook_file> -i <inventory_file> --ask-vault-pass
# load the password from a plain text file
ansible-playbook <playbook_file> -i <inventory_file> -vault-password-file <vault_password_file>
# load the password from an executable (python) script
ansible-playbook <playbook_file> -i <inventory_file> -vault-password-file <script>
```

It is possible to view the content of an encrypted file using:
```bash
ansible-vault view <inventory_file>
```
It is possible to create an encrypted file using:
```bash
ansible-vault create <inventory_file>
```


## Dynamic Inventory

Normally the inventory file is just a plain text file, but it can also be a 
python script. In this case the data returned by the script can be dynamic.

The basic example of a inventory script it to return the content statically:
```python
#!/usr/bin/env python

import json

def get_inventory_data():
    return {
        "database": {
            "host": ["db_server"],
            "vars": {
                "ansible_ssh_pass": "Passw0rd",
                "ansible_ssh_host": "127.0.0.1"
            }
        },
        "web": {
            "host": ["web_server"],
            "vars": {
                "ansible_ssh_pass": "Passw0rd",
                "ansible_ssh_host": "127.0.0.1"
            }
        }
    }

if __name__ == "__main__":
    data = get_inventory_data
    print(json.dumps(data))
```
The `#!/usr/bin/env python` is needed by Ansible to understand how to run it.

The `get_inventory_data` function is static in this case but it can also 
retrieve the data from a database or some other source.

It is then possible to use the script in this way:
```bash
ansible-playbook <playbook_file> -i <inventory_script>
```

Further Ansible can call the python script with two different arguments:
- `--list`:\
should output a json encoded dictionary of all groups (the example above).
- `--host <host>`:\
should output the list of variables available for that particular host.

The script above can be updated as follow:
```python
#!/usr/bin/python

import argparse
import json

def get_inventory_data():
    return {
        "database": {
            "host": ["db_server"],
            "vars": {
                "ansible_ssh_pass": "Passw0rd",
                "ansible_ssh_host": "127.0.0.1"
            }
        },
        "web": {
            "host": ["web_server"],
            "vars": {
                "ansible_ssh_pass": "Passw0rd",
                "ansible_ssh_host": "127.0.0.1"
            }
        }
    }

def read_cli_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('--list', action='store_true')
    parser.add_argument('--host', action='store')
    return parser.parse_args()


if __name__ == "__main__":
    args = read_cli_args()
    data = get_inventory_data
    if args and args.list:
        print(json.dumps(data))
    elif args.host:
        # not sure why wants to return an empty list in this case, 
        # maybe it is just as example
        print(json.dumps({'_meta': {'hostvars': []}}))
```

