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

## CSV Templating: Adding Custom Ansible facts.d Facts
Somewhat confusingly, Ansible-CMDB's official docs call these 'host local facts".

Say you've put in the hard work to have your hosts start registering custom facts to be collected into Ansible-CMDB.

Once these facts start showing up in Ansible properly, and are reported back by the Ansible Setup module, you can use their JSON path to define them into your CSV templates.

We'll use this example block from my [patch_facts](https://github.com/kfiresmith/patch_facts) script, which creates custom OS-reported patch status facts.d fact files on Linux hosts:
```json
        "ansible_local": {
            "os_patch_status": {
                "all_updates": "9",
                "date_collected": "2021-06-21T18:30-04:00",
                "eol": "false",
                "errata_support": "true",
                "os_updates_broken": "false",
                "security_updates": "0"
            }
        },
```
To add these facts to Ansible-CMDB's CSV template, we'll have to refer to them using the following syntax:

```python
  {"title": "All Updates",       "id": "all_updates",       "visible": True, "field": lambda h: h['ansible_facts'].get('ansible_local',{}).get('os_patch_status',{}).get('all_updates', '')},
  {"title": "Security Updates",       "id": "security_updates",       "visible": True, "field": lambda h: h['ansible_facts'].get('ansible_local',{}).get('os_patch_status',{}).get('security_updates', '')},
  {"title": "Is EOL?",       "id": "eol_status",       "visible": True, "field": lambda h: h['ansible_facts'].get('ansible_local',{}).get('os_patch_status',{}).get('eol', '')},
  {"title": "OS Updates Broken?",       "id": "os_updates_broken",       "visible": True, "field": lambda h: h['ansible_facts'].get('ansible_local',{}).get('os_patch_status',{}).get('os_updates_broken', '')},
```

What we see here is that we're drilling down the dot-delimited path to the JSON data we need from the collected custom facts, using a series of `.get()` methods, one for each segment of the JSON path down to the value we need.

So `ansible_local.os_patch_status.eol` is 3 segments deep within the JSON block. 
