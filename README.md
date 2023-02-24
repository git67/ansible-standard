## ansible-standard 
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

#### it's up to you, to install pre-commit -> https://pre-commit.com/#install
#### a simple configuration example for hook ansible-lint is included (.pre-commit-config.yaml) 

#### for update the collection tree or the python3 venv (if used) you cant do this by the following commands

```
pip install -Ur requirements.pip
ansible-galaxy collection install -r requirements.yml
```
