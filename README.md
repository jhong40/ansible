# ansible
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
```
