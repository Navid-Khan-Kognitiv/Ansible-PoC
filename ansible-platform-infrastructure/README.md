# Ansible roles and playbooks set
In this repository you can find playbooks which apply configurations, create users and also install software in the servers.

## Directory structure
 ```
kognitiv-Ansible
|--.ssh            <----- You can find the IP, private key and ssh configs of the managed hosts
|
|--collections     <----- Ansible install modules which you will use for the playbooks and roles (lib/modules)
|
|--group_vars      <----- Used to create files containing variables set for a groups of hosts
|
|--host_vars       <----- Used to create files containing variables set for a specific host
|
|--playbooks       <----- Playbooks which you will run over the servers, for each server you will have a playbook with its name.
|
|--roles           <----- Used to save the created roles
|
|--vault           <----- Store the secrets in encyrpted files for ansible-vault
|   |
|   |--sudo.yaml   <----- This file will have your default user and sudo password if it needed and it's ignored by git.
|
|--ansible.conf    <----- Base configuration of Ansible for this project
|
|--vaultpass       <----- This contains the password for encrypt and decrypt secrets of ansible-vault (included in .gitignore)
 ```


## Requeriments

- SSH Private key in the path ~/.ssh/id_rsa
  To generate it you can run in terminal

  ```
    ssh-keygen
  ```

- Add vaultpass file in the root of project (this contains the secret password used to encrypt/decrypt secrets)
  

- Create encrypted fiele /vault/sudo.yam with your username and sudo password adding _vault_ansible_user_ and 
  _vault_ansible_sudo_pass variables_ with yours.
  
  ```
    ansible-vault create vault/sudo.yaml
  ```

## Add a new host
To add a new managed host, you need to modify the __inventory__ file (in the root of project) adding the host name and then set the connection parameters in .ssh/config file. In this last, you must specify the hostname equals to the inventory file defining the parameter Hostname with its IP or FQDN, user and private key. For example:

```
Host smn-dev-vnet-bastion-vm
    Hostname 20.200.77.117
    User azureuser
    IdentityFile ~/.ssh/smn-dev-vnet-bastion-vm
```

## How to run a playbook
If you need run a existent playbook for an added server it's very siple, as we told previously in directory structure you can find one playbook for each server added to inventory with the same name of the server. 
To run the playbok for my_server_name you can execute in a terminal.

```
ansible-playbook playbooks/my_server_name.yaml
```

If you need to overwrite the SSH user and ask sudo password, you can ovewrite it via command parameters for SSH user (-u).

```
ansible-playbook playbooks/my_server_name.yaml -u my_alternative_user
```

## Defining secrets
Ansible-vault allow to create a file in the specified path and encrypt it with the password provided in the file _vaultpass_ at the root of the project. Here you can define values and ansible will proccess it as variables which you can retrive when you are runing a playbook. Also you can encrypt a whole folder or a file with sensitive data and decrypt the content when you copy these files to the remote host.

### Create secret

Following the previous examples, you can create a secret file for _my_server_name_ an define a variable to call from the playbooks of this server. As a good practice, it is recommended that all variables defined within a vault file begin with the  _vault__ prefix, in this way you will know that this variable is a secret when is used in a playbook.

```
ansible-vault create vault/my_server_name.yaml


```

### View/edit existing secret


```
ansible-vault edit vault/my_server_name.yaml
ansible-vault view vault/my_server_name.yaml
```

### Encrypt existing file

```
ansible-vault encrypt my_file.rar
```

### How to use these secrets
To apply these secrets to a playbok you have two ways. The first, which is used in these repo, is to add the file of secrets in a vars_files inside the playbook specifying the path of the file. In the example you can see the definition in the playbook  __playbooks/smn-dev-vnet-bastion-vm.yml__, and the second way is adding -e paramenter to the __ansible-playbook__ command.


#### Specifying the file with secrets in playbook file
```
- name: Running playbook for smn-dev-vnet-bastion-vm
  hosts: smn-dev-vnet-bastion-vm
  become: true
  connection: ssh
  gather_facts: true
  vars_files:
    - ../vault/smn-dev-vnet-bastion-vm.yaml          <------- The file is specified here
```

#### Specifying the file with secrets in playbook run command

```
ansible-playbook -e "@vault/smn-dev-vnet-bastion-vm.yaml"
```
## Usefull commands

### Search a module
In this case I'm searchin modules with user in its name
```
ansible-doc -l |grep -i user
```

### Search documentation of a module
If you need help to know wich parameters use in a module, you can search in modules documentation. In this case I'm getting help about the user module.


ansible-doc user
Useful commands


### Test connection to my_host

```
ansible my_host -m ping
```

### Ad-Hoc executions
If you need to run no automation task, you can run ad-hoc commands wich will be executed one time. You can use any ansible module with this command.

In this example is executing __uname -r__ command to know which kernel is installed in the remote host.

```
ansible my_host -a 'uname -r'
```

Here copy file to my_host or insert content into it

```
ansible my_host -m copy -a 'src=/etc/mi-motd dest=/etc/motd'
ansible my_host -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd'
```
