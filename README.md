# ansible
```
ansible all -m shell -a 'df -h'        # all - all host, -m module shell, -a argument 'df -h'
ansible node-1 -m shell -a 'df -h'     # node-1
ansible all -m shell -a 'hostname'     # list hostname
ansible all -m ping                   # module ping
ansible node-1 -m ping -o             #-o : Summarize the output into one line per node.
ansible node-1 -m shell -a 'which screen'   # check screen pkg installed or not
ansible node-1 -b -m dnf -a 'name=screen state=latest'   # dnf like yum, apt, install screen pkg, -b: become
ansible node-1 -m setup                # retrieve info from node -> ansible_{variable}

ansible web -m uri -a 'url=http://localhost/ return_content=yes'   # check url
ansible node-1 -m debug -a 'msg={{ hoge | default("abc") }}'       # debug mode -> abc
ansible node-1 -e 'str=abc' -m debug -a 'msg="{{ str | upper }}"'  # -> ABC
ansible node-1 -m debug -a 'msg="{{ [5, 1, 10] | min }}"'          # -> 1

ansible node-1 -b -m yum -a 'name=httpd state=present'
ansible node-1 -b -m systemd -a 'name=httpd state=started enabled=yes'
ansible node-1 -b -m systemd -a 'name=httpd state=stopped enabled=yes'    


ansible-lint lint_ng_playbook.yml                                 # lint .yml
ansible-lint -L                                                   # checking rule
ansible-lint -T                                                   # checking tag
ansible-lint -x unnamed-task -x yaml lint_ng_playbook.yml         # excloude unnamed task to make it ok


ansible --version

ansible [core 2.11.5] 
  config file = /root/.ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /opt/kata-materials/ansible/lib/python3.8/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /opt/kata-materials/ansible/bin/ansible
  python version = 3.8.10 (default, Sep 28 2021, 16:10:42) [GCC 9.3.0]
  jinja version = 3.0.2
  libyaml = True

cat ~/.ansible.cfg
[defaults]
inventory          = /root/inventory_file
host_key_checking  = False
force_color        = True
forks              = 1

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null

(ansible) ubuntu $ cat ~/inventory_file
[web]
node-1 ansible_host=172.17.0.2
node-2 ansible_host=172.17.0.3
node-3 ansible_host=172.17.0.4
[all:vars]
ansible_user=centos
ansible_ssh_private_key_file=/root/automation-keypair.pem
(ansible) ubuntu $

ansible web --list-hosts  # list node-1,2,3
ansible-doc -l       # list installed ansible modules

```
```
ansible-playbook my.yml --syntax-check      # only validate syntax, not exe
ansible-playbook mydir/**/* --syntax-check  # check all the files under mydir
ansible-playbook my.yml --check             # look for "changed:", dry-run, if wrong package like ngingx (nginx), it will detect
ansible-playbook my.yml --check --diff      # show which package will be installed in "changed"
ansible-playbook my.yml --limit host1,host2 # specific host or group
ansible-playbook my.yml --limit webservers  # specific group
ansible-playbook my.yml --tag=database      
ansible-playbook my.yml --tag=start,stop    # will stop the svc, and start according to the order in the playbook

ansible-playbook my.yml -v                  # verbose mode -vv: tell task filename,line number, -vvv

ANSIBLE_LOG_PATH=/myfolder/myansible.log    # capture ansilbe output to a file
ANSIBLE_DEBUGT=1                            # generates lot of output.. better > a file to look

```

```
- ansible.builtin.foo:
    bar: baz
  check_mode: true                         # true: run in checkmode even if --check is not used, false: run normal even if commandline with --check

- ansible.builtin.debug:
    msg: "Hello World"                     # msg or var (not both)
    var: foobar
    verbose: 3
    var: ansible_facts.mounts | selectattr('mount', 'equalto', '/') | map(attribute='size_availabe')  # select the root mounts section

- name: After version 2.7 both O(msg) and O(fail_msg) can customize failing assertion message
  ansible.builtin.assert:
    that:
      - my_param <= 100
      - my_param >= 0
    fail_msg: "'my_param' must be between 0 and 100"
    success_msg: "'my_param' is between 0 and 100"

- name: Example using fail and when together
  ansible.builtin.fail:
    msg: The system may not be provisioned according to the CMDB status.
  when: cmdb_status != "to-be-staged"

- name: restart DB
  meta: flush_handlers    # clear_facts, clear_host_errors, end_host, end_play, flush_handlers, noop, refresh_inventory, reset_connection

- name: wait for port 80 to be open
  wait_for:
    timeout: 10
    port: 80
    state: started        # after the port 80 is open, then check 80 accessibiliity


```
## Ansible debugger 
[https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_debugger.html](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_debugger.html#available-debug-commands)

```
(ansible) ubuntu $ cat z.yml 
---
- name: Install packages on managed hosts
  hosts: node-1
  tasks:
    - name: Install packages
      package:
        name: 
          - nginx
          - wget
          - zenmap
        state: present
      debugger: always
      become: yes

(ansible) ubuntu $ ansible-playbook z.yml 
/opt/kata-materials/ansible/lib/python3.8/site-packages/paramiko/pkey.py:82: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "cipher": algorithms.TripleDES,
/opt/kata-materials/ansible/lib/python3.8/site-packages/paramiko/transport.py:253: CryptographyDeprecationWarning: TripleDES has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.TripleDES and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  "class": algorithms.TripleDES,

PLAY [Install packages on managed hosts] ******************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [node-1]

TASK [Install packages] ***********************************************************************************************************************
fatal: [node-1]: FAILED! => {"changed": false, "failures": ["No package zenmap available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
[node-1] TASK: Install packages (debug)> p task.args
{'name': ['nginx', 'wget', 'zenmap'], 'state': 'present'}
[node-1] TASK: Install packages (debug)> p task.args['name']
['nginx', 'wget', 'zenmap']
[node-1] TASK: Install packages (debug)> task.args['name'][-1]='nmap'
[node-1] TASK: Install packages (debug)> p task.args['name']
['nginx', 'wget', 'nmap']
[node-1] TASK: Install packages (debug)> r
changed: [node-1]
[node-1] TASK: Install packages (debug)>
```
