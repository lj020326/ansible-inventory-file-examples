
# Example 4: Multiple YAML inventories with role based groups

In the prior [Example 3](../example3/README.md), we found the method to merge multiple YAML inventories with the merged results observing intended or expected behavior.

Now we will look to apply plays that can target machines in the merged inventory with role-based groups.

E.g., the following scenario will discuss a simple NTP client/server based playbook to apply across the merged inventory. 

## Overview

In this example there are 2 networks located at 2 sites resulting in 4 YAML inventory files, with hierarchy diagrammed as follows:

```mermaid
graph TD;
    A[all] --> B[dmz]
    A[all] --> C[internal]
    B --> D["site1<br>dmz/site1.yml"]
    B --> E["site2<br>dmz/site2.yml"]
    C --> F["site1<br>internal/site1.yml"]
    C --> G["site2<br>internal/site2.yml"]
```


For each of the 4 inventory files, the following group/host hierarchy will be implemented:

```mermaid
graph TD;
    A[all] --> C[hosts]
    A[all] --> D[children]
    C --> I["web-[dmz|internal]-q1-s[1|2].example.int"]
    C --> J["web-[dmz|internal]-q2-s[1|2].example.int"]
    D --> E[rhel7]
    D --> F[environment_qa]
    D --> G["location_site[1|2]"]
    D --> H["network_[dmz|internal]"]
    E --> K[hosts]
    K --> L["web-[dmz|internal]-q1-s[1|2].example.int"]
    K --> M["web-[dmz|internal]-q2-s[1|2].example.int"]
    F --> N[hosts]
    N --> O["web-[dmz|internal]-q1-s[1|2].example.int"]
    N --> P["web-[dmz|internal]-q2-s[1|2].example.int"]
    G --> Q[hosts]
    Q --> R["web-[dmz|internal]-q1-s[1|2].example.int"]
    Q --> S["web-[dmz|internal]-q2-s[1|2].example.int"]
    H --> T[hosts]
    T --> U["web-[dmz|internal]-q1-s[1|2].example.int"]
    T --> W["web-[dmz|internal]-q2-s[1|2].example.int"]
```


Each site.yml inventory will be setup similar to the following with the "[dmz|internal]" and "[1|2]" regex patterns evaluated for each of the 4 cases:

```yaml
all:
  hosts:
    admin-[dmz|internal]-q1-s[1|2].example.int: 
      trace_var: site[1|2]/admin-[dmz|internal]-q1-s[1|2].example.int
      foreman: <94 keys>
    admin-[dmz|internal]-q2-s[1|2].example.int: 
      trace_var: site[1|2]/admin-[dmz|internal]-q1-s[1|2].example.int
      foreman: <94 keys>
    app-[dmz|internal]-q1-s[1|2].example.int: 
      trace_var: site[1|2]/app-[dmz|internal]-q1-s[1|2].example.int
      foreman: <94 keys>
    app-[dmz|internal]-q2-s[1|2].example.int: 
      trace_var: site[1|2]/app-[dmz|internal]-q1-s[1|2].example.int
      foreman: <94 keys>
    web-[dmz|internal]-q1-s[1|2].example.int:
      trace_var: site[1|2]/web-[dmz|internal]-q1-s[1|2].example.int
      foreman: <94 keys>
    web-[dmz|internal]-q2-s[1|2].example.int:
      trace_var: site[1|2]/rhel7/web-[dmz|internal]-q2-s[1|2].example.int
      foreman: <94 keys>
  children:
    rhel6:
      vars:
        trace_var: dmz/site1/rhel6
      hosts:
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
    rhel7:
      vars:
        trace_var: site[1|2]/rhel7
      hosts:
        admin-[dmz|internal]-q2-s[1|2].example.int: {}
        app-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q2-s[1|2].example.int: {}
        web-[dmz|internal]-q1-s[1|2].example.int: {}
        web-[dmz|internal]-q2-s[1|2].example.int: {}
    environment_qa:
      vars:
        trace_var: site[1|2]/environment_qa
      hosts:
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q2-s[1|2].example.int: {}
        web-[dmz|internal]-q1-s[1|2].example.int: {}
        web-[dmz|internal]-q2-s[1|2].example.int: {}
    location_site[1|2]:
      vars:
        trace_var: site[1|2]/location_site[1|2]
      hosts:
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q2-s[1|2].example.int: {}
        web-[dmz|internal]-q1-s[1|2].example.int: {}
        web-[dmz|internal]-q2-s[1|2].example.int: {}
    network_[dmz|internal]:
      vars:
        trace_var: site[1|2]/network_[dmz|internal]
      hosts:
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        admin-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q1-s[1|2].example.int: {}
        app-[dmz|internal]-q2-s[1|2].example.int: {}
        web-[dmz|internal]-q1-s[1|2].example.int: {}
        web-[dmz|internal]-q2-s[1|2].example.int: {}
    ungrouped: {}

```

Each of the respective inventory files:

* [dmz/site1 inventory](./inventory/dmz/site1.yml)
* [dmz/site2 inventory](./inventory/dmz/site2.yml)
* [internal/site1 inventory](./inventory/internal/site1.yml)
* [internal/site2 inventory](./inventory/internal/site2.yml)



## Define NTP inventory groups

For the ntp playbook/role to work on both servers and clients, we will define the 'ntp_server' and 'ntp_client' groups to correctly scope the machines to be applied.

For each network/site, there will be 2 __ntp servers__ resulting in a total of 8 hosts to be targeted for the 'ntp-server' play/role application.

Specifically, the 'ntp_server' group configuration will be applied to the following 8 'admin' machines (2 host instances for each specific network/site):

```output
admin-dmz-q1-s1.example.int
admin-dmz-q2-s1.example.int
admin-dmz-q1-s2.example.int
admin-dmz-q2-s2.example.int
admin-internal-q1-s1.example.int
admin-internal-q2-s1.example.int
admin-internal-q1-s2.example.int
admin-internal-q2-s2.example.int
```


The 'ntp-client' group will include all linux machines for the respective environment.
In this case, the environment will be defined with the existing test environment group named 'environment_test'.

Now we can define the YAML groups to be used by the 'ntp' playbook/role as follows:

[inventory/dmz/ntp.yml](./inventory/dmz/ntp.yml):
```yaml
all:
  children:
    ntp_server:
      hosts:
        admin-q1-dmz-s1.example.int: {}
        admin-q2-dmz-s1.example.int: {}
        admin-q1-dmz-s2.example.int: {}
        admin-q2-dmz-s2.example.int: {}
    ntp_client:
      children:
        environment_test: {}
```

[inventory/internal/ntp.yml](./inventory/internal/ntp.yml):
```yaml
all:
  children:
    ntp_server:
      hosts:
        admin-q1-internal-s1.example.int: {}
        admin-q2-internal-s1.example.int: {}
        admin-q1-internal-s2.example.int: {}
        admin-q2-internal-s2.example.int: {}
    ntp_client:
      children:
        environment_test: {}
```

The 'ntp_client' group is defined with the children group of 'environment_test'.  

Note that the 'ntp_client' group includes the 8 admin machines already included in the 'ntp_server' group.  This overlap can be addressed by making sure that the 'ntp_server' group is excluded for the respective plays that only mean to target the 'ntp_client' machines.  This will be demonstrated in the following verifications section. 

We will now run through several ansible CLI tests to verify that the correct machines result for each respective limit used.

### Test 1: Target all ntp servers

```shell
ansible -i ./inventory/ ntp_server  -m debug -a var=trace_var,group_names
admin-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}

```

This is as expected.


### Test 2: Target all ntp clients

As mentioned earlier, the 'ntp_clients' group is defined using the children group of 'environment_test'.  The following ansible debug command excludes the 'ntp_server' hosts from that set such to target only the non-'ntp-server' hosts.

```shell
ansible -i ./inventory/ ntp_client,\!ntp_server  -m debug -a var=trace_var,group_names
app-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/app-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
app-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/app-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
app-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/app-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
app-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/app-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/web-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/web-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
app-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/app-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/app-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}

```

We see that all the ntp-client machines and no ntp-server machines result. 
This is as expected.

## Testing Conclusion

The 2 test results demonstrate that we can safely target the ntp_server and ntp_client machines with the appropriate group targets.

We now seek to apply those filters in the next ntp playbook section.


## NTP group variables

### Environment specific variable settings

Each network-site environment has a different gateway with a respective unique ipv4 address.  The gateway ipv4 address is used to derive the network mask for each respective environment, which in turn is used to properly derive the ntp allow/restrict network mask setting used for each ntp server.

Set up the gateway_ipv4 variable for each network/site.

We do this by adding a section for each site group (location_site[1|2]) for the appropriate variable settings to be added to the respective inventory ntp.yml as follows.

[inventory/dmz/ntp.yml](./inventory/dmz/ntp.yml)
```yaml
all:
  children:
    ntp_server:
      hosts:
        admin-q1-dmz-s1.example.int: {}
        admin-q2-dmz-s1.example.int: {}
        admin-q1-dmz-s2.example.int: {}
        admin-q2-dmz-s2.example.int: {}
    ntp_client:
      children:
        environment_test: {}
    location_site1:
      vars:
        trace_var: dmz/ntp/location_site1
        gateway_ipv4: 112.112.0.1
        gateway_ipv4_network_cidr: 112.112.0.0/16
    location_site2:
      vars:
        trace_var: dmz/ntp/location_site2
        gateway_ipv4: 221.221.0.1
        gateway_ipv4_network_cidr: 221.221.0.0/16

```

[inventory/internal/ntp.yml](./inventory/group_vars/network_internal.yml)
```yaml
all:
  children:
    ntp_server:
      hosts:
        admin-q1-internal-s1.example.int: {}
        admin-q2-internal-s1.example.int: {}
        admin-q1-internal-s2.example.int: {}
        admin-q2-internal-s2.example.int: {}
    ntp_client:
      children:
        environment_test: {}
    location_site1:
      vars:
        trace_var: internal/ntp/location_site1
        gateway_ipv4: 192.168.112.1
        gateway_ipv4_network_cidr: 192.168.112.0/16
    location_site2:
      vars:
        trace_var: internal/ntp/location_site2
        gateway_ipv4: 192.168.221.1
        gateway_ipv4_network_cidr: 192.168.221.0/16
```

### Verify that the correct gateway_ipv4 setting appears for each ntp server.

```shell
ansible -i ./inventory/ -m debug -a var=group_trace_var,gateway_ipv4 ntp_server
admin-q1-dmz-s1.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.112.1')"
}
admin-q2-dmz-s1.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.112.1')"
}
admin-q1-dmz-s2.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.221.1')"
}
admin-q2-dmz-s2.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.221.1')"
}
admin-q1-internal-s1.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.112.1')"
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.112.1')"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.221.1')"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "group_trace_var,gateway_ipv4": "('group_vars/ntp_client.yml', '192.168.221.1')"
}

```


### Group vars for play/role specific settings

Set up group variables for the respective ntp groups.

[inventory/group_vars/ntp_server.yml](./inventory/group_vars/ntp_server.yml)
```yaml
---

## ntp-server configs
## ref: https://github.com/geerlingguy/ansible-role-ntp
ntp_timezone: America/New_York
ntp_area: 'us'

ntp_tinker_panic: true

ntp_allow_networks:
  - "{{ gateway_ipv4_network_cidr }}"

#ntp_servers:
#  - "{{ gateway_ip4 }} prefer iburst"

ntp_servers:
  - 0{{ '.' + ntp_area if ntp_area else '' }}.pool.ntp.org iburst xleave
  - 1{{ '.' + ntp_area if ntp_area else '' }}.pool.ntp.org iburst xleave
  - 2{{ '.' + ntp_area if ntp_area else '' }}.pool.ntp.org iburst xleave
  - 3{{ '.' + ntp_area if ntp_area else '' }}.pool.ntp.org iburst xleave

ntp_peers: |
  [
    {% for host in groups['ntp_server'] | difference([inventory_hostname]) %}
    {{ hostvars[host].ansible_host }},
    {% endfor %}
  ]

ntp_local_stratum_enabled: yes

ntp_leapsectz_enabled: yes

ntp_log_info:
  - measurements
  - statistics
  - tracking

ntp_cmdport_disabled: no

## used for variable-to-inventory trace/debug
group_trace_var: group_vars/ntp_server.yml

```

[inventory/group_vars/ntp_client.yml](./inventory/group_vars/ntp_client.yml)
```yaml
---

## ntp-client configs
## ref: https://github.com/geerlingguy/ansible-role-ntp
ntp_timezone: America/New_York

ntp_tinker_panic: yes

ntp_servers: |
  [
    {% if ansible_default_ipv4.address|d(ansible_all_ipv4_addresses[0]) is defined %}
    {% if groups['ntp_server'] is defined %}
    {% for server in groups['ntp_server'] %}
    {% for network in hostvars[server].ntp_allow_networks|d([]) %}
    {% if ansible_default_ipv4.address|d(ansible_all_ipv4_addresses[0]) | ansible.utils.ipaddr('network') %}
    "{{ hostvars[server].ansible_host }}",
    {% endif %}
    {% endfor %}
    {% endfor %}
    {% endif %}
    {% endif %}
  ]

ntp_cmdport_disabled: yes

## used for variable-to-inventory trace/debug
group_trace_var: group_vars/ntp_client.yml

```


## NTP Playbook

[playbook.yml](./playbook.yml):
```yaml
---

- name: "Setup ntp servers"
  hosts: ntp_server
  tags:
    - bootstrap-ntp
    - bootstrap-ntp-server
  become: yes
  roles:
    - role: geerlingguy.ntp

- name: "Setup ntp clients"
  hosts: ntp_client,!ntp_server
  tags:
    - bootstrap-ntp
    - bootstrap-ntp-client
  become: yes
  roles:
    - role: geerlingguy.ntp

```


### Show the resulting ntp_servers variable

Run for groups 'ntp_server,\&network_internal'
```shell
ansible -i ./inventory/ -m debug -a var=ntp_servers ntp_server,\&network_internal
admin-q1-internal-s1.example.int | SUCCESS => {
    "ntp_servers": [
        "0.us.pool.ntp.org iburst xleave",
        "1.us.pool.ntp.org iburst xleave",
        "2.us.pool.ntp.org iburst xleave",
        "3.us.pool.ntp.org iburst xleave"
    ]
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "ntp_servers": [
        "0.us.pool.ntp.org iburst xleave",
        "1.us.pool.ntp.org iburst xleave",
        "2.us.pool.ntp.org iburst xleave",
        "3.us.pool.ntp.org iburst xleave"
    ]
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "ntp_servers": [
        "0.us.pool.ntp.org iburst xleave",
        "1.us.pool.ntp.org iburst xleave",
        "2.us.pool.ntp.org iburst xleave",
        "3.us.pool.ntp.org iburst xleave"
    ]
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "ntp_servers": [
        "0.us.pool.ntp.org iburst xleave",
        "1.us.pool.ntp.org iburst xleave",
        "2.us.pool.ntp.org iburst xleave",
        "3.us.pool.ntp.org iburst xleave"
    ]
}

```


## Debug host vars using groups to target sets of hosts

Run debug using a group defined set of hosts.

### Specify role & network/location groups

Run for groups 'ntp_server,\&network_internal'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names ntp_server,\&network_internal
admin-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}

```

Run for groups 'ntp_server,\&location_site2'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names ntp_server,\&location_site2
admin-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}

```

### Specify network/location groups

Run for group 'network_internal'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names network_internal
admin-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}
app-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}
app-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/app-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/app-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}

```

Run for group 'location_site1'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names location_site1
admin-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
app-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/app-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
app-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/app-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
admin-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/admin-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}
app-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
app-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/app-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}

```

Run for group(s) matching multiple groups 'ntp_server,&network_dmz'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names ntp_server,\&network_dmz
admin-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}

```

Run for group(s) matching multiple groups 'location_site2,&ntp_server'
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names location_site2,\&ntp_server
admin-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/admin-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel7'])"
}
admin-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel6'])"
}
admin-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/admin-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'ntp_server', 'rhel7'])"
}

```

## Limits

### Limit to specific hosts in a group

```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names ntp_server -l admin-q1-dmz-s1.example.int
admin-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/admin-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'ntp_server', 'rhel6'])"
}

```

### Limit hosts in the role-based group

Run for the role-based group 'ntp_client' with a specified limit
```shell
ansible -i ./inventory/ -m debug -a var=trace_var,group_names ntp_client -l web-*
web-q1-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q1-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q2-dmz-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site1/web-q2-dmz-s1.example.int', ['environment_test', 'location_site1', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q1-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/web-q1-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q2-dmz-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('dmz/site2/web-q2-dmz-s2.example.int', ['environment_test', 'location_site2', 'network_dmz', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q1-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s1.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site1/web-q2-internal-s1.example.int', ['environment_test', 'location_site1', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q1-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q1-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}
web-q2-internal-s2.example.int | SUCCESS => {
    "trace_var,group_names": "('internal/site2/web-q2-internal-s2.example.int', ['environment_test', 'location_site2', 'network_internal', 'ntp_client', 'rhel7'])"
}

```