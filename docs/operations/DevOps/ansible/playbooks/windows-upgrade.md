
##### Windows automatically upgrade

```yaml linenums="1"
- name:  Windows Update
  hosts: wserver.merox.cloud
  gather_facts: false
  tasks:
    - name: Running Windows Update
      win_updates:
        category_names: ['SecurityUpdates','CriticalUpdates','UpdateRollups', 'Updates', 'DefinitionUpdates']
        reboot: yes

```