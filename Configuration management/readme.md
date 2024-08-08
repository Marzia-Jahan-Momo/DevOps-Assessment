## Configuration management

### Ansible command can display all ansible_ configuration for a host
    sudo vim /etc/ansible/hosts
    [nodes]
    node1 ansible_host=192.168.17.142 ansible_user=momo
    node2 ansible_host=192.168.17.143 ansible_user=momo

    ansible nodes -m setup



#### momo@master:/etc/ansible$ ansible nodes -m setup -a 'filter=ansible_*_ipv4'
```
node2 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "192.168.17.143",
            "alias": "ens33",
            "broadcast": "192.168.17.255",
            "gateway": "192.168.17.2",
            "interface": "ens33",
            "macaddress": "00:0c:29:fd:57:b0",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "192.168.17.0",
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
node1 | SUCCESS => {
    "ansible_facts": {
        "ansible_default_ipv4": {
            "address": "192.168.17.142",
            "alias": "ens33",
            "broadcast": "192.168.17.255",
            "gateway": "192.168.17.2",
            "interface": "ens33",
            "macaddress": "00:0c:29:2d:9e:a3",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "192.168.17.0",
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
```
---
### Configuring a cron job that runs logrotate on all machines every 10 minutes between 2h - 4h.


#### logrotate.yml

```
---
- name: Configure logrotate cron job
  hosts: all
  become: yes
  tasks:
    - name: Add logrotate cron job
      cron:
        name: "Logrotate job"
        minute: "*/10"
        hour: "2-4"
        job: "/usr/sbin/logrotate /etc/logrotate.conf"

```


#### cat ansible.cfg
```
[defaults]
inventory = /etc/ansible/hosts
sudo_user = root
timeout = 30
```


#### cat hosts
```
[nodes]
192.168.17.142
192.168.17.143

```

#### ansible-playbook logrotate.yml
```
PLAY [Configure logrotate cron job] **************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
ok: [192.168.17.142]
ok: [192.168.17.143]

TASK [Add logrotate cron job] ********************************************************************************************
changed: [192.168.17.142]
changed: [192.168.17.143]

PLAY RECAP ***************************************************************************************************************
192.168.17.142             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.17.143             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```



#### momo@master:/etc/ansible$ ansible all -m shell -ba "crontab -l"
```
192.168.17.143 | CHANGED | rc=0 >>
#Ansible: Logrotate job
*/10 2-4 * * * /usr/sbin/logrotate /etc/logrotate.conf
192.168.17.142 | CHANGED | rc=0 >>
#Ansible: Logrotate job
*/10 2-4 * * * /usr/sbin/logrotate /etc/logrotate.conf

```

---
### Deploying ntpd package to the following 3 servers


#### momo@master:/etc/ansible$ cat deploy_ntpd.yml
```
---
- name: Deploy and configure ntpd
  hosts: all
  become: yes
  tasks:
    - name: Install ntp package
      apt:
        name: ntp
        state: present

    - name: Configure /etc/ntp.conf
      copy:
        dest: /etc/ntp.conf
        content: |
          tinker panic 0
          restrict default kod nomodify notrap nopeer noquery
          restrict -6 default kod nomodify notrap nopeer noquery
          restrict 127.0.0.1
          restrict -6 ::1
          server   192.168.0.252 minpoll 4 maxpoll 8
          server   192.168.0.251 minpoll 4 maxpoll 8
          server   192.168.0.0 # local clock
          fudge    192.168.0.0 stratum 10
          driftfile /var/lib/ntp/drift
          keys     /etc/ntp/keys

    - name: Ensure ntpd is started and enabled
      service:
        name: ntp
        state: started
        enabled: yes

```


#### ansible-playbook deploy_ntpd.yml
```
PLAY [Deploy and configure ntpd] *****************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************
ok: [192.168.17.142]
ok: [192.168.17.143]

TASK [Install ntp package] ***********************************************************************************************
changed: [192.168.17.143]
changed: [192.168.17.142]

TASK [Configure /etc/ntp.conf] *******************************************************************************************
changed: [192.168.17.142]
changed: [192.168.17.143]

TASK [Ensure ntpd is started and enabled] ********************************************************************************
ok: [192.168.17.143]
ok: [192.168.17.142]

PLAY RECAP ***************************************************************************************************************
192.168.17.142             : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
192.168.17.143             : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

# Check NTP service status

#### momo@master:/etc/ansible$ ansible all -m shell -a "systemctl status ntp"
```
192.168.17.143 | CHANGED | rc=0 >>
● ntp.service - Network Time Service
     Loaded: loaded (/lib/systemd/system/ntp.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-08-08 02:59:06 +06; 3min 40s ago
       Docs: man:ntpd(8)
    Process: 11512 ExecStart=/usr/lib/ntp/ntp-systemd-wrapper (code=exited, status=0/SUCCESS)
   Main PID: 11518 (ntpd)
      Tasks: 2 (limit: 7570)
     Memory: 1.3M
        CPU: 141ms
     CGroup: /system.slice/ntp.service
             └─11518 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 130:137

আগস্ট 08 02:59:09 node2 ntpd[11518]: Soliciting pool server 2402:1f00:8000:800::36bb
আগস্ট 08 02:59:11 node2 ntpd[11518]: Soliciting pool server 91.189.91.157
আগস্ট 08 02:59:12 node2 ntpd[11518]: Soliciting pool server 185.125.190.56
আগস্ট 08 02:59:13 node2 ntpd[11518]: Soliciting pool server 185.125.190.58
আগস্ট 08 02:59:14 node2 ntpd[11518]: Soliciting pool server 185.125.190.57
আগস্ট 08 02:59:15 node2 ntpd[11518]: Soliciting pool server 2620:2d:4000:1::40
আগস্ট 08 03:00:16 node2 ntpd[11518]: Soliciting pool server 2001:678:8::123
আগস্ট 08 03:00:23 node2 ntpd[11518]: Soliciting pool server 2620:2d:4000:1::41
আগস্ট 08 03:01:21 node2 ntpd[11518]: Soliciting pool server 2406:da18:6d1:a600::be00:5
আগস্ট 08 03:01:32 node2 ntpd[11518]: Soliciting pool server 2620:2d:4000:1::3f
192.168.17.142 | CHANGED | rc=0 >>
● ntp.service - Network Time Service
     Loaded: loaded (/lib/systemd/system/ntp.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-08-08 02:59:19 +06; 3min 27s ago
       Docs: man:ntpd(8)
    Process: 10398 ExecStart=/usr/lib/ntp/ntp-systemd-wrapper (code=exited, status=0/SUCCESS)
   Main PID: 10405 (ntpd)
      Tasks: 2 (limit: 7570)
     Memory: 1.4M
        CPU: 102ms
     CGroup: /system.slice/ntp.service
             └─10405 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 130:137

আগস্ট 08 02:59:24 node1 ntpd[10405]: Soliciting pool server 91.189.91.157
আগস্ট 08 02:59:25 node1 ntpd[10405]: Soliciting pool server 185.125.190.56
আগস্ট 08 02:59:26 node1 ntpd[10405]: Soliciting pool server 185.125.190.58
আগস্ট 08 02:59:27 node1 ntpd[10405]: Soliciting pool server 185.125.190.57
আগস্ট 08 02:59:28 node1 ntpd[10405]: Soliciting pool server 2620:2d:4000:1::40
আগস্ট 08 03:00:28 node1 ntpd[10405]: Soliciting pool server 2001:678:8::123
আগস্ট 08 03:00:34 node1 ntpd[10405]: Soliciting pool server 2620:2d:4000:1::41
আগস্ট 08 03:01:33 node1 ntpd[10405]: Soliciting pool server 2406:da18:6d1:a600::be00:5
আগস্ট 08 03:01:41 node1 ntpd[10405]: Soliciting pool server 2620:2d:4000:1::3f
আগস্ট 08 03:02:39 node1 ntpd[10405]: Soliciting pool server 2606:4700:f1::123

```

#### momo@master:/etc/ansible$ cat deploy_nagios.yaml
```
---
- name: Deploy Nagios templates for NTP servers
  hosts: all
  become: yes
  vars:
    ntp_servers:
      - { name: "node1", ip: "192.168.17.142" }
      - { name: "node2", ip: "192.168.17.143" }
  tasks:
    - name: Install Nagios
      apt:
        name: nagios4
        state: present
      become: yes

    - name: Ensure Nagios servers directory exists
      file:
        path: /usr/local/nagios/etc/servers
        state: directory
        owner: nagios
        group: nagios
        mode: '0755'

    - name: Create Nagios host configuration
      copy:
        dest: "/usr/local/nagios/etc/servers/{{ item.name }}.cfg"
        content: |
          define host {
            host_name                      {{ item.name }}
            address                        {{ item.ip }}
            check_command                  check-ping
            active_checks_enabled          1
            passive_checks_enabled         1
          }
          define service {
            service_description            ntp_process
            host_name                      {{ item.name }}
            check_command                  check_ntp
            check_interval                 10
          }
      with_items: "{{ ntp_servers }}"
      notify:
        - Restart Nagios

  handlers:
    - name: Restart Nagios
      service:
        name: nagios
        state: restarted


``` 

#### momo@master:/etc/ansible$ ssh momo@192.168.17.142 "cat /usr/local/nagios/etc/servers/node1.cfg"

```
define host {
  host_name                      node1
  address                        192.168.17.142
  check_command                  check-ping
  active_checks_enabled          1
  passive_checks_enabled         1
}
define service {
  service_description            ntp_process
  host_name                      node1
  check_command                  check_ntp
  check_interval                 10
}
momo@master:/etc/ansible$ ssh momo@192.168.17.143 "cat /usr/local/nagios/etc/servers/node1.cfg"
define host {
  host_name                      node1
  address                        192.168.17.142
  check_command                  check-ping
  active_checks_enabled          1
  passive_checks_enabled         1
}
define service {
  service_description            ntp_process
  host_name                      node1
  check_command                  check_ntp
  check_interval                 10
}

```
