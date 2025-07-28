# Usage Instructions

This is how you should run the playbooks (the .yml files): 

```bash
ansible-playbook -i inventory name_of_the_playbook.yml --ask-vault-password
```

As you can see, the following options were used:

- `-i` option is used for linking the inventory file to this playbook run
- `--ask-vault-password` option is used in order to provide the password for the vault file the playbook is linked to (check the `vars_files` inside any playbook present inside this repo to understand how to link the vault to the playbook)

Another option that you can use is the `--check` option. What this does is that it lets you run the playbook without making any changes on the slave node.
