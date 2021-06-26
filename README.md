# ansible-cmdb-examples
Additional examples for customizing Ansible-CMDB

## Rationale
Ansible-CMDB is widely used, critically important, and largely abandoned by it's maintainer AFAICT :/
So instead of submitting another ignored PR to build out the documentation examples a bit more, I'll put them here.
The goal is to build something of a missing manual for Ansible-CMDB users, especially those of us who don't do much coding.


## CSV Templating: Adding Ansible Inventory Host and Group Variables ("hostvars")
[Ansible-CMDB Documentation for Host and Group Variables](https://ansible-cmdb.readthedocs.io/en/latest/usage/#host-and-group-variables)

[Ansible-CMDB Documentation for Creating Custom Templates](https://ansible-cmdb.readthedocs.io/en/latest/usage/#custom-templates)

In order to add inventory variables such as `dtap`, `comment`, or `ext_id`, use `host['hostvars'].get`, like so:

```python
  {"title": "DTAP", "id": "dtap", "visible": True, "field": lambda h: host['hostvars'].get('dtap', '')},
```

To gain on-par capability to the html_fancy template, add this block:

```python
  {"title": "Groups", "id": "groups", "visible": True, "field": lambda h: h.get('groups', '')},
  {"title": "DTAP", "id": "dtap", "visible": True, "field": lambda h: host['hostvars'].get('dtap', '')},
  {"title": "External ID", "id": "ext_id", "visible": True, "field": lambda h: host['hostvars'].get('ext_id', '')},
  {"title": "Comment", "id": "comment", "visible": True, "field": lambda h: host['hostvars'].get('comment', '')},
```
**Note:** The Groups column is a bit different.  It uses `h.get('var', '')`

This format works for any host or group vars declared in your inventory file or within your host_vars/group_vars folder structure in the same folder as your inventory file.


