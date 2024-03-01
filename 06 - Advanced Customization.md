# Advanced Customization

- [Custom Modules](#custom-modules)
- [Custom Plugins](#custom-plugins)

---

## Custom Modules

A modules are the elements that perform the actions in the tasks list.
There are already a lot modules for:
- system
- commands
- files
- database
- cloud
- etc.

Nevertheless it can happen that none of these modules really satisfies the needs,
therefore it is possible to create custom ones.
 
Develop a custom module means develop a python script, for example a custom 
module that debugs by adding the time as prefix looks like this:
```python
#!/usr/bin/python

# utilities import
try:
    import json
except ImportError:
    import simplejson as json

# helper import
from ansible.module_utils.basic import AnsibleModule
import time
import sys

DOCUMENTATION = '''
---
this is some documentation for this module.
'''

EXAMPLES = '''
# Example
custom_debug:
  msg: 'The debug message'
'''


def main(): 
    # definition of the Ansible module
    module = AnsibleModule(
        argument_spec = dict(
            # definition of the parameter (with required and type information)
            msg=dict(required= True, type='str')
        )
    )

    # get the parameter
    msg = module.params['msg']

    # DO NOT PRINT ANYTHING ON SCREEN
    try:
        # successful case
        module.exit_json(changed=True, msg='%s - %s' % (time.strftime("c"), msg))
    except:
        # unsuccessful case
        module.fail_json(msg="failed debugging")

if __name__ == '__main__':
    main()
```
This file can be either placed under the modules directory or under the library 
one in a role definition.

The module can then be used in this way:
```yaml
- name: Custom Module
  hosts: localhost
  tasks:
    - name: Custom Debug
      custom_debug:
        msg: 'The debug message'
```

More info about how to develop a module can be found on github, by browsing the
[Modules' source code](https://github.com/ansible/ansible/tree/devel/lib/ansible/modules)
or in the 
[Developing modules documentation](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html).

## Custom Plugins

There are several plugins used by Ansible like:
- Action plugin to invoke modules
- Connection plugin to establish communication with hosts
- Filter plugin to manipulate data
- Lookup plugin to load data from external sources
- Strategy plugin to control flow and execution of play
- Callback plugin to handle events
- etc.

It is possible to create custom plugins, like an average filter, by creating
a file called `average.py` with:
```python
def average(numbers):
    '''Find the average of a list of numbers
    '''
    return sum(numbers) / float(len(numbers))

# Mandatory class name 
class FilterModule(object):
    ''' Query filter '''

    # Mandatory method the return the filter name and function
    def filters(self):
        return {
            'average': average
        }
```
It is then possible to use the in this way:
```yaml
- name: Custom Filter plugin
  hosts: localhost
  vars:
    ints:
      - 10
      - 20
      - 30
      - 40
      - 50
  tasks:
    - debug:
        msg: "{{ ints | average }}"
```
To make it available is also needed to export the folder where is located:
```bash
export ANSIBLE_FILTER_PLUGINS=/path/to/the/plugins/folder
```

It is also possible to develop the other plugins types, a good starting point
is copy and existing one and adapt it with the help of the documentation.

