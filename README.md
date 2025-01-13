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

```
