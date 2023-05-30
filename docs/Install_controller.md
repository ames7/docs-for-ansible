## Prerequisites 

### RHEL 9 system 
   - Minimum 16GB Ram 
   - Minimum 4 CPUs
   - Minimum 40 GB Disk
      - If installing Postgresql DB on this node its recommended to have a 150GB+ disk

### Ports Needed 
- 22 (SSH)
- 80 (HTTP)
- 443 (HTTPS)
- 5432 (Postgresql)
- 27199 (Receptor)

### Attach AAP Subscription 

1. Register your system: 
   ```
   subscription-manager register
   ```

 > ℹ️ Note
 > 
 > If you have [Simple Content Access](https://access.redhat.com/articles/simple-content-access) enabled you dont need to attach a subscription  

1. Repositories are disabled by configuration on RHEL 9, so we need to enable them: 
   ```
   subscription-manager config --rhsm.manage_repos=1
   ```
2. Check to make sure the manage_repos setting changed: 
   ```
   subscription-manager config | grep -i manage_repos 
   ```
3. Find the pool ID for AAP subscription: 
   ```
   subscription-manager list --available --all | grep "Ansible Automation Platform" -B 3 -A 6
   ```
   If you have issues finding the repo make sure its enabled: 
   ```
   subscription-manager repos --enable ansible-automation-platform-2.3-for-rhel-9-x86_64-rpms
   ```
4. Attach the subscription: 
   ```
   subscription-manager attach --pool=<pool_id>
   ```
5. Make sure the subscription attached: 
   ```
   subscription-manager list --consumed
   ```
### Download Installer files 
  
1. Head over to [https://access.redhat.com/downloads/content/480](https://access.redhat.com/downloads/content/480)
2. Download the Ansible Automation Platform 2.3 Setup file 
3. Untar the file: 
   ```
   tar xvzf ansible-automation-platform-setup-2.3-2.tar.gz
   ```

### Create a Registry Service Account 

1. Log into [Registry Service Account Management Application](https://access.redhat.com/terms-based-registry/)
2. Click the New Service Account button 
3. Enter a name for the account (It will be prepended with a fixed, random string)
4. Enter a description so later you will know what the account is for...
5. Click Create 
6. Click on the Service Account you just created (these are used to login to registry.redhat.io to pull images/containers)
   1. Note the username ( ie. XXXXX|username ) 
   2. Note the password 

### Edit the Installer Inventory File 

> Use this example inventory file for a Standalone Automation Controller with an internal database 

```
[automationcontroller]
controller.acme.org

[all:vars]
admin_password='<password>'
pg_host=''
pg_port='5432'
pg_database='awx'
pg_username='awx'
pg_password='<password>'
pg_sslmode='prefer'  # set to 'verify-full' for client-side enforced SSL

registry_url='registry.redhat.io'
registry_username='<registry username>'
registry_password='<registry password>'
```
#### Automation Controller Variable

- For the [automationcontroller] variable, you can use a FQDN or a IP address or both, whichever your host can resolve
- If you use both an IP Address and a FQDN make sure to add the 'ansible_host' variable to one of them 
```
[automationcontroller]
 controller.acme.org 

[automationcontroller:vars]
 ansible_host=0.0.0.0
```
- If you are running the install from the controller host make sure to add a 'ansible_connection=local' variable
```
[automationcontroller]
 controller.acme.org 

[automationcontroller:vars]
 ansible_connection=local
```
- If you are running on an EC2 host you will need to add an variable for the 'ansible_user' and 'ansible_become'
```
[automationcontroller]
 controller.acme.org 

[automationcontroller:vars]
 ansible_user=ec2-user 
 ansible_become=true
```

#### All:Vars Variable 

```
[all:vars]
admin_password='<password>'
pg_host=''
pg_port='5432'
pg_database='awx'
pg_username='awx'
pg_password='<password>'
pg_sslmode='prefer'  # set to 'verify-full' for client-side enforced SSL

registry_url='registry.redhat.io'
registry_username='<registry username>'
registry_password='<registry password>'
```

- For the 'admin_password' variable, this is the password that you will use to login into the UI after the install 
- For the 'pg_host' variable, in this installation scenario we will leave this blank as we are not installing the database separately 
- For the 'pg_port' variable, enter the postgresql port (5432)
- For the 'pg_database' variable, this is what you want to name the postgresql database 
- For the 'pg_username' variable, this is the user you will login into the database with 
- For the 'pg_password' variable, this is the password you will login into the databse with 
- For the 'pg_sslmode' variable, unless you are configuring client-side SSL leave this as is 
- For the 'registry_url' variable, leave this as registry.redhat.io 
- For the 'registry_username' variable, this is the username you created earlier in the [Create a Registry Service Account] 
- For the 'registry_password' variable, this is the password you created earlier in the [Create a Registry Service Account] 

## Running the installer Script 

- Run the setup.sh script 
  ```
  ./setup.sh
  ```

### Verify Installation of Automation Controller 

- Go to the IP address/FQDN of the controller in your inventory file 
  ``` 
  https://controller.acme.org
  ```
- Login using the username 'admin' and the 'admin_password' you set in the inventory file 

## 🎉 Congrats! You installed Automation Controller! 🎉