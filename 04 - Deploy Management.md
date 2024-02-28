# Deploy Management

- [Handlers](#handlers)
- [Async Actions](#async-actions)
- [Deploy Strategy](#deploy-strategy)
- [Error Handling](#error-handling)

---

## Handlers

Handlers allow to define action to restart all the web services and associate
it with a task that updates the configuration file.\
The idea is to automatically trigger a restart whenever a configuration file 
is updated, eliminating the manual intervention.

Basically handlers are tasks triggered by specific events or notifications.\
They are defined inside playbooks and executed when notified by a task.

```yaml
- name: Play Handler
  hosts: localhost
  tasks:
    - name: Copy application code
      copy: 
        src: app_code
        dst: /opt/applicaion/
        notify: Restart Application Service

  handlers:
    - name: Restart Application Service
      service:
        name: application_service
        status: restarted
```
If there are multiple tasks notifying the same handler during a playbook run,
the handler will only called once at the end of the play.j

## Async Actions

It is possible to perform an async tast, as for an health check, that must be
run for a certain period of time (for example 6 minutes) using the async
property and polling the status every minute:
```yaml
- name: Play Async
  hosts: localhost
  tasks:
    - command: /opt/health_check.sh
      async: 360
      poll: 60 # default value 10 seconds
```
If there is a second service to monitor, by putting this new monitoring task 
after the first one, it will result in scheduling the second only after the
first one has finished, that means after 6 minutes:
```yaml
- name: Play Async
  hosts: localhost
  tasks:
    - command: /opt/service_health_check.sh
      async: 360
      poll: 60
    - command: /opt/database_health_check.sh
      async: 360
      poll: 60
```
It is possible to run both of them in parallel by setting poll to 0, so that
Ansible will proceed to the next task:
```yaml
- name: Play Async
  hosts: localhost
  tasks:
    - command: /opt/service_health_check.sh
      async: 360
      poll: 0
    - command: /opt/database_health_check.sh
      async: 360
      poll: 0
```
at this point Ansible will exit immediately without waiting the needed time.
To make it wait, it is necessary to register the output to a variable and
check the status using the async_status module:
```yaml
- name: Play Async
  hosts: localhost
  tasks:
    - command: /opt/service_health_check.sh
      async: 360
      poll: 0
      register: service_result
    - command: /opt/database_health_check.sh
      async: 360
      poll: 0
      register: database_result
    - name: Check status of tasks
      async_status: jid={{ service_result.ansible_job.id }} 
      register: job_result
      until: job_result.finished
      retries: 30
```

## Deploy Strategy

While running a playbook there are different strategies that can be used
to manage the tasks execution in the hosts:
- linear
- free
- batch
- custom

### linear

Each task must complete successfully on all the hosts before moving to the 
next task, this is the default one.
```yaml
- name: Deploy Strategy
  strategy: linear # optional
  hosts: all_servers
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - name: third
       # ...
```

### free

Each task run independently on each host, the next task is started as soon as
possible without waiting the other host to be at the same progress level.
```yaml
- name: Deploy Strategy
  strategy: free
  hosts: all_servers
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - name: third
       # ...
```

### batch

Defines the number of hosts on which the playbook should run and then proceed
with the next batch.
```yaml
- name: Deploy Strategy
  serial: 3 # also percentage like 20%
  hosts: all_servers
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - name: third
       # ...
```

### custom

Define a custom strategy.
    

### Fork

The maximum number of host managed by Ansible is defined by the `forks` 
parameter in the `ansible.cfg` that by default is five.


## Error Handling

When only one host is configured and a task fails, the playbook will exit
immediately.

On the other hand, if more than one host is configured and a task fails for a
particular host, the playbook takes that host out of the list and continues
with the other.\
The default behaviour is to try to complete on as must host as possible.

It is possible to configure Ansible to stop the execution on all the hosts
if one fails by setting the property `any_errors_fatal`:
```yaml
- name: Error Handling
  hosts: all_servers
  any_errors_fatal: true # default false
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - name: third
       # ...
```

On the other hand, if a task is not crucial for the all system, it is possible
to ignore its failure by setting the `ignore_errors` property:
```yaml
- name: Error Handling
  hosts: all_servers
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - name: third
       # ...
      ignore_errors: true # default false
```

It is also possible to explicitly fail if some errors are found:
```yaml
- name: Error Handling
  hosts: all_servers
  tasks:
    - name: fist
       # ...
    - name: second
       # ...
    - command: cat /var/log/server.log
      register: command_output
      failed_when: "'ERROR' in command_output.stdout"
```

