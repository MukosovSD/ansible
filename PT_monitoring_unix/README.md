# Ansible role for Linux Expert Pack

## Quick start
1. Create system users on all target hosts that can elevate privileges using `su` or `sudo`.
2. Download and save all role files in some directory on the system where **Ansible 2.9** or later is installed.
3. Add the target hosts addresses (IP or FQDN) to which you want to apply the role to the `./hosts` file
4. Set PT MaxPatrol SIEM IP or FQDN address (see **Target agents configuration** section below).
5. Define username which will be used to connecting to the hosts (option `ansible_ssh_user` in the `./hosts` file).
6. Apply the role with the following command:
    ```
    ansible-playbook -i hosts --ask-become-pass -k ./lep_src_ansible.yml
    ```

## Local host usage
1. Create local system users that can elevate privileges using `su` or `sudo`.
2. Download and save all role files in some directory on the system where **Ansible 2.9** or later is installed.
3. Set PT MaxPatrol SIEM IP or FQDN address (see **Target agents configuration** section below).
4. Apply the role with the following command:
    ```
    ansible-playbook --connection=local --ask-become-pass ./local_lep_src_ansible.yml
    ```
## Supported Linux distributions
- Canonical Ubuntu 16.04, 18.04, 20.04, 22.04
- CentOS 7, 8
- Oracle Linux 7, 8
- Red Hat Enterprise Linux 7, 8
- Debian 9, 10, 11, 12

## Directory tree and files purpose
```
.
├── ansible.cfg
├── hosts
├── lep_src_ansible.yml
├── local_lep_src_ansible.yml
├── README.md
└── roles
    └── lep-src
        ├── defaults
        │   └── main.yml
        ├── files
        │   └── packages
        │       └── <...>
        ├── tasks
        │   ├── configure
        │   │   ├── audispd-plugins.yml
        │   │   ├── auditd.yml
        │   │   ├── rsyslogd.yml
        │   │   ├── syslog-ng.yml
        │   │   ├── syslog.yml
        │   │   └── system-wide.yml
        │   ├── install
        │   │   ├── without_repos
        │   │   │   ├── audispd-plugins.yml
        │   │   │   ├── audit.yml
        │   │   │   ├── misc.yml
        │   │   │   └── rsyslog.yml
        │   │   └── with_repos
        │   │       ├── audispd-plugins.yml
        │   │       ├── auditd.yml
        │   │       ├── rsyslogd.yml
        │   │       └── system-wide.yml
        │   └── main.yml
        ├── templates
        │   ├── auditd_00-siem.rules.j2
        │   ├── rsyslog_10-siem.conf.j2
        │   └── syslog-ng_10-siem.conf.j2
        └── vars
            ├── main.yml
            └── siem_agents.yml
```

- `./ansible.cfg` -- options defining Ansible software behavior
- `./hosts` -- target hosts list (see [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html))
- `./lep_src_ansible.yml` -- Ansible playbook which will apply the role
- `./roles/lep-src/vars/main.yml` -- containing the main options used in the role
- `./roles/lep-src/vars/siem_agents.yml` -- target SIEM agent configuration
- `./roles/lep-src/templates/` -- template files in Jinja2 format. Mainly used for configuration files modifying on target hosts
- `./roles/lep-src/tasks/install/` -- tasks that do software installation
- `./roles/lep-src/tasks/configure/` -- tasks that do software configuration
- `./roles/lep-src/files/packages/` -- directory containing packages used for software installation, if repositories are not available



## Configuration
Options can be specified in one of the following ways, in order of priority, where 1 is the lowest priority:
1. Globaly in the `./roles/lep-src/vars/main.yml` file.
2. Globaly in the `./hosts` file (see [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)).
3. For specific hosts in the `./hosts` file. For examle:
    ```
    ...
    10.0.214.15 use_repos=false
    ...
    ```
### Target agents configuration
This Ansible role allows to configure one or more agents for all hosts or for groups of hosts. To distribute hosts and agents into groups, the concept of "facility"
and the same variable name are used. There is always the "default" facility exists, it is used if no facility is defined for the host(s).

There can be the following relationships between the described entities:
- Each target host can be associated with only one facility
- One agent can be associated with several facilities
- Any facility may not be associated with any of the hosts
- If no facility is defined for the host, the `default` facility will be used

To configure agent addresses and facilities, the following files should be edited:
- `./roles/lep-src/vars/siem_agents.yml` -- file where defines the address(es) of agent(s) and their facility matching
- `./hosts` -- list of hosts and their facility membership

Below are examples of configuring agents for different cases.

#### Case #1: One agent for all hosts
**./roles/lep-src/vars/siem_agents.yml:**
```
facilities:
  default:
    - { address: 'mpxagent01.example.com', transport: 'udp', port: '514' }
```

**./hosts:**
```
centos7.example.com
centos8.example.com
ubuntu1604.example.com
ubuntu1804.example.com
```

#### Case #2: Messages from all hosts are duplicated for several agents
**./roles/lep-src/vars/siem_agents.yml:**
```
facilities:
  default:
    - { address: 'mpxagent01.example.com', transport: 'udp', port: '514' }
    - { address: 'mpxagent02.example.com', transport: 'udp', port: '514' }
```

**./hosts:**
```
centos7.example.com
centos8.example.com
ubuntu1604.example.com
ubuntu1804.example.com
```

#### Case #3: Messages from different hosts are sent to different agents or groups of agents
**./roles/lep-src/vars/siem_agents.yml:**
```
facilities:
 
 default:
   - { address: 'mpxagent01.example.com', transport: 'udp', port: '514' }
   - { address: 'mpxagent02.example.com', transport: 'udp', port: '514' }
 tatooine:
   - { address: 'mpxagent01.alpha.example.com', transport: 'udp', port: '514' }
   - { address: 'mpxagent01.beta.example.com', transport: 'tcp', port: '514' }
 coruscant:
   - { address: 'mpxagent01.example.com', transport: 'udp', port: '21514' }
   - { address: 'mpxagent03.example.com', transport: 'udp', port: '21514' }
   - { address: 'mpxagent04.example.com', transport: 'udp', port: '20514' }
```

**./hosts:**
```
[RHEL_based]
oracle7.example.com
oracle8.example.com facility=tatooine
centos7.example.com facility=coruscant
centos82.example.com
 
[Debian]
debian9.example.com
debian10.example.com
 
[Debian:vars]
facility=coruscant
```

- Messages from `oracle7.example.com` and `centos82.example.com` will be sent to agents in the `default` facility
- Messages from `oracle9.example.com` will be sent to the `tatooine` facility
- Messages from `centos7.example.com`, `debian9.example.com` and `debian10.example.com` will be sent to the `coruscant` facility
- The agent `mpxagent01.example.com` is present in two facilities at once: `default` and `coruscant`


### Options
| Option | Possible values | Default | Description |
| --- | --- | --- | --- |
| upload_local_packages | true of false | false | If `true`, then the required packages will be uploaded to the target systems when they are not available in the repositories |
| auditd_write_logs | true or false   | false   | If `true`, then audit logs will be saved to file system, otherwise not |
| modify_etc_hosts | true of false | false | If `true`, the contents of the hosts file will be changed. This deletes localhost records |
| auditd_config | FS path | /etc/audit/auditd.conf | Auditd main configuration file |
| auditd_rules_dir | FS path | /etc/audit/rules.d/ | Directory contanis Auditd rules files |
| auditd_siem_ruleset | filename | 00-siem.rules | Auditd rule set for PT MaxPatrol SIEM |
| auditd_rules_bak_name | filename | rules_backup.tar | Archive name where current Auditd rules will be placed |
| audispd_plugin_config_v2<br>audispd_plugin_config_v3 | FS path | /etc/audisp/plugins.d/syslog.conf<br>/etc/audit/plugins.d/syslog.conf | Auditsp-plugins configuration file path |
| syslog_options | string | 'rsyslogd\|syslog-ng' | Acceptable syslog daemon name |
| rsyslog_config | FS path | /etc/rsyslog.conf | rsyslog main configuration file |
| rsyslog_siem_config | FS path | /etc/rsyslog.d/10-siem.conf | File where PT MaxPatrol SIEM options for rsyslog will be placed |
| rsyslog_options | YAML list | --- | rsyslog configruation options and values shich will be modified |
| syslog_ng_config | FS path | /etc/syslog-ng/syslog-ng.conf | syslog-ng main configuration file |
| syslog_ng_siem_config | FS path | /etc/syslog-ng/10-siem.conf | File where PT MaxPatrol SIEM options for syslog-ng will be placed |
| linux_supported | YAML list | --- | List of supported Linux distributions to which the role can be applied |
| packages_dirs | YAML dict | --- | Directories and distribution matching for packages uploading |
