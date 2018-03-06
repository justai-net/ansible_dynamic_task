# Generate a vmware_guest module task using a Jinja2 template.
Running the playbook, 
``` yaml
---
- hosts: localhost
  gather_facts: no
  vars_files:
  - vars/vars.yml

  tasks:
  - name: Generate build tasks for virtual machines
    template:
      src: templates/task_template.j2
      dest: './tasks/{{ item.name }}.yml'
    with_items:
    - "{{ nodes }}"
```
templates/task_template.j2
```yaml
---
- name: Building {{ item.name }}
  vmware_guest:
    validate_certs: no
    name: {{ item.name }}
    template: {{ item.os }}
    disk:
{% for items in item.harddrive %}
    - size_gb: {{ items.size_gb }}
      type: {{ items.type }}
      datastore: {{ items.datastore }}
{% endfor %}
    networks:
{% for nic in item.network %}
{% if nic.type == 'static' %}
    - name: {{ nic.name }}
      type: {{ nic.type }}
      ip: {{ nic.ip }}
      netmask: {{ nic.netmask }}
      gateway: {{ nic.gateway }}
{% else %}
    - name: {{ nic.name }}
      type: {{ nic.type }}
{% endif %}
{% endfor %}
    hardware:
      memory_mb: {{item.hardware.memory_mb }}
      num_cpus: {{ item.hardware.num_cpus }}
      scsi: paravirtual
```

### Will output
tasks/justai-web01.yml
``` yaml
---
- name: Building justai-web01
  vmware_guest:
    validate_certs: no
    name: justai-web01
    template: Centos7
    disk:
    - size_gb: 50
      type: thin
      datastore: datastore2
    - size_gb: 30
      type: thin
      datastore: datastore2
    networks:
    - name: BridgedNetwork
      type: dhcp
    - name: VM Network
      type: static
      ip: 192.168.30.111
      netmask: 255.255.255.0
      gateway: 192.168.30.1
    hardware:
      memory_mb: 1024
      num_cpus: 2
      scsi: paravirtual
```

tasks/justai-db01.yml

``` yaml
---
- name: Building justai-db01
  vmware_guest:
    validate_certs: no
    name: justai-db01
    template: Centos7
    disk:
    - size_gb: 50
      type: thin
      datastore: datastore2
    - size_gb: 30
      type: thin
      datastore: datastore2
    networks:
    - name: BridgedNetwork
      type: dhcp
    - name: VM Network
      type: static
      ip: 192.168.30.112
      netmask: 255.255.255.0
      gateway: 192.168.30.1
    hardware:
      memory_mb: 1024
      num_cpus: 2
      scsi: paravirtual
```
