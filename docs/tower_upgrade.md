## Migrating from Ansible Tower 3.8 to AAP 2.3

>ℹ️ Notes before install

- Tower 3.8 requires an M4.large or larger instance on AWS 

- AAP requires an M4.xlarge or larger 

#### Tower 3.8.6-2 Installs
  - Tower 3.8.6 (Controller 3.8)
  - Ansible 2.9.27 
  - Postgresql 10.23 
  - Python 3.6.8
 
#### AAP 2.3 Installs
   - AAP 2.3.8
   - Ansible 2.14.3
   - Postgresql 13.7
   - Python 3.9.14

## Backup Old Tower Environment

- Run setup script with backup option
```
./setup.sh -b
```
- Copy backup file to new host

## Migrate Tower Virtual Environments

- Check for Virtual Environments
``` 
awx-manage list_custom_venvs

Example output:

Discovered Virtual Environments:
/var/lib/awx/venv/ansible/
```

- Check the Associations of Virtual Environments
```
awx-manage custom_venv_associations </path/to/venv>

Virtual Environments Associations:
inventory_sources: []
job_templates: []
organizations: []
projects: []
```

- Export Custom Environment to See Packages Installed 
```
awx-manage export_custom_venv /path/to/venv

Virtual environment contents:
adal==1.2.2
ansible==2.9.27
apache-libcloud==2.5.0
appdirs==1.4.3
applicationinsights==0.11.9
```

### Playbook
Here is an example playbook you can use to compare list of packages from virt envs to Ansible 2.9 Execution Environment
```
---
- name: Review custom virtualenvs and pull requirements
  hosts: localhost
  become: true
  tasks:
- name: Include venv role
  ansible.builtin.include_role:
    name: redhat_cop.ee_utilities.virtualenv_migrate
```

- Pulls a list of packages from each custom Python virtual environments present on the Ansible Automation Platform 1.2 environment
- Compares the package lists from the previous step with the package list of the minimal execution environment
- Create a new custom execution environment that uses the Ansible-2.9 execution environment as the base and including the missing dependencies from the list in the previous step
  
### Inventory File 
```
[tower]
ansibletower.example.com
ansible_ssh_private_key_file=/path/to/example.pem


[all:vars]
###############################################################################
# Required configuration variables for migration from venv -> EE              #
###############################################################################

# The default URL location to the execution environment (Default Ansible 2.9)
# If you want to use the newest Ansible base, change to: ee-minimal-rhel8:latest
venv_migrate_default_ee_url="registry.redhat.io/ansible-automation-platform-21/ee-29-rhel8:latest"

# User credential for access to venv_migrate_default_ee_url
registry_username='myusername'
registry_password='mypassword'
```

## Install Ansible Tower 3.8 (on new host)
- Get the Ansible Tower installer 
 ```
 https://releases.ansible.com/ansible-tower/setup/ansible-tower-setup-latest.tar.gz 
 ```

 - Untar the file 
 ```
 tar xvzf ansible-tower-setup-latest.tar.gz 
 ```

 - Cd into the directory 
```
 cd ansible-tower-setup-<tower_version>
```

 - Find your pool ID 
```
 subscription-manager list --available --all | grep "Ansible Automation Platform" -B 3 -A 6 
```

- Attach the subscription 
 ```
 subscription-manager attach --pool=<pool_id> 
 ```

- Check to make sure it attached 
 ```
 subscription-manager list --consumed
 ```
- Copy inventory file over from previous Tower host 

- Run the setup script using the restore function 
```
./setup.sh -r 
```

## Upgrade Tower to AAP

> ℹ️ Note:
>  If you are running the upgrade on the same host you will need to subscribe the box to the new AAP repo using subscription-manager 
    ```
     subscription-manager repos --enable ansible-automation-platform-2.3-for-rhel-8-x86_64-rpms
    ```
> If you get the message 'Repositories disabled by configuration' you need to allow subscription-manager to manage the repository
     ```
     subscription-manager config --rhsm.manage_repos=1
     ```
> Verify the manage_repos value has changed to 1 
     ```
     subscription-manager config | grep -i manage_repos 
     ```
> Then re-run the enable command above ⬆️

### Upgrade to AAP
- Download new version of Ansible Automation Installer 
```
https://access.redhat.com/downloads/content/480/
```

- Untar the file 
```
tar zxvf ansible-automation-platform-setup-2.3.2.tar.gz
```

- Change to the new directory 
```
cd ansible-automation-platform-setup-2.3.2
```

> ℹ️ Note: You can run the setup script to generate a new inventory file based on the old one (The installer will fail with the error: "The installer has detected that you are using an inventory format from a version prior to 4.0. We have created an example inventory based on your old style inventory.")
    > ```
    > ./setup.sh
    > ```

- Fix the inventory file to include the new names/variables for AAP 2.3 
  - Tower changes to Automation controller 
  ```
  [tower] --> [automationcontroller]
  ```
- Add registry credentials to inventory (create a registry account if you do not have one)
    - Follow this guide to [create a registry account](https://access.redhat.com/RegistryAuthentication#creating-registry-service-accounts-6) if you dont have one 
    ```
    registry_url='registry.redhat.io' 
    registry_username='myusername'
    registry_password='mypassword'
    ```
- Run the setup script with your new inventory 
  ```
  ./setup.sh -i inventory
  ```
- After the install has finished go to your new AAP instance address to check that it is accessible 
  ```
  https://<controllerhostfqdn>
  ```

## Congrats you are now Upgraded to AAP 2.3! 🎉

