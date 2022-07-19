## ansible-standard   jj 
#### clone these repository for local use
```
git clone https://github.com/git67/ansible-standard <your_working_dir>
```


#### create/update your inventory file
###### (look into ansible.cfg for reference to this file)
```
vi inventory/hosts
```

#### source environment
```
source .env
alias
```

#### create ansible environment at managed nodes 
###### (keep in mind, you have to know the initial root password of your managed nodes)
```
_ri p_configure_ansible.yml
```

#### now you can work complete interaktivly
```
_ra <playbook>
```

#### it's up to you, to install pre-commit -> https://pre-commit.com/#install
#### a simple configuration example for hook ansible-lint is included (.pre-commit-config.yaml) 
