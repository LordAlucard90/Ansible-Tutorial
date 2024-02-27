# Jinja

## Content

- [Jinja2](#jinja2)
- [General](#general)
- [Ansible](#ansible)
- [Templates](#templates)
- [File Lookup](#file-lookup)

## Jinja2

[Jinja2](https://jinja.palletsprojects.com/en/3.1.x/) 
templates it is a templating engine for python

## General

Some of the features for strings are (given `name=John doe`):
- Substitute:\
`The name is {{ name }}` becomes `The name is John doe` 
- Upper:\
`The name is {{ name | upper }}` becomes `The name is JOHN DOE` 
- Lower:\
`The name is {{ name | lower }}` becomes `The name is john doe` 
- title:\
`The name is {{ name | title }}` becomes `The name is John Doe` 
- Replace:\
`The name is {{ name | replace("John", "Jane") }}` becomes `The name is Jane doe` 
- Default:\
`The name is {{ name | default("John") }}` becomes `The name is John` 
(if `name` not defined) 

Some of the features for lists and sets are:
- Min:\
`{{ [1, 2, 3] | min }}` becomes `1` 
- Max:\
`{{ [1, 2, 3] | max }}` becomes `3` 
- Unique:\
`{{ [1, 2, 3, 2, 1] | unique }}` becomes `[1, 2, 3]` 
- Union:\
`{{ [1, 2, 3, 4] | union([4, 5]) }}` becomes `[1, 2, 3, 4, 5]` 
- Intersect:\
`{{ [1, 2, 3, 4] | intersect([4, 5]) }}` becomes `[4]` 
- Random:\
`{{ 100 | random }}` becomes a random value
- Join:\
`{{ ["John", "Doe"] | join(" ") }}` becomes `John Doe` 
- First:\
`{{ ["John", "Doe"] | first }}` becomes `John` 
- Last:\
`{{ ["John", "Doe"] | last }}` becomes `Doe` 

It is possible to iterate over a list in this way:
```
{% for i in [1, 2, 3] %}
  Cur nunber is {{ i }}
{% endfor %}
```
and use conditions:
```
{% for i in [1, 2, 3] %}
  {% if i >= 0 %}
    Cur posiive nunber is {{ i }}
  {% endif %}
{% endfor %}
```

## Ansible

Ansible has extended Jinja to adapt it to its context.

Some of the features for files are:
- Base name:\
`{{ "/etc/hosts" | basename }}` becomes `hosts` 
- Windows base name:\
`{{ "c:\windows\hosts" | win_basename }}` becomes `hosts` 
- Windows split drive:\
`{{ "c:\windows\hosts" | win_splitdrive }}` becomes `["c:", "\windows\hosts"]` 
- First:\
`{{ "c:\windows\hosts" | win_splitdrive | first }}` becomes `c:` 
- Last:\
`{{ "c:\windows\hosts" | win_splitdrive | last }}` becomes `\windows\hosts` 


For IP addresses:
- IP addresses:\
`{{ 127.0.0.1 | ipaddr }}` becomes `127.0.0.1`


A complete playbooks' filters list can be found in the
[Playbooks' filters documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html)

## Templates

It is possible to define files templates like `index.html.j2`:
```html
<!DOCTYPE html>
<html>
  <body>
        This is {{ inventory_hostname }} Server.
  </body>
</html>
```
and use them in the playbooks:
```yaml
- name: Play Templates
  hosts: all_servers
  tasks:
    - name: Copy index.html to servers
      template: 
        src: index.html.j2
        dest: /var/www/nginx-defaults/index.html
```
It is possible to improve this process by adding filters to format nemas or
specify default values.

The templates can be stored in the `templates` folder.

## File Lookup

The lookup plugin is used to load data from a file, for example, given a csv
file with this content:
```csv
Hostname;Password
Target01;Secret-1
Target02;Secret-2
```
It is possible to retrieve the password for an host using:
```
{{ lookup('csvfile', 'target01 file=/path/to/file.csv delimiter=;') }}
```
where:
- `csvfile` is the file type
- `target01` is the host
- `file=...` is the source file
- `delimiter` is the fields delimiter char

Other lookup types are:
- ini
- dns
- MongoDB 
- etc..

More info can be found in the documentation pages 
[Lookups](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_lookups.html),
[Playbook Lookups](https://docs.ansible.com/ansible/latest/plugins/lookup.html)
and
[Lookup plugins](https://docs.ansible.com/ansible/latest/collections/index_lookup.html).


