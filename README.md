# RDLabo
## Requirements - Ansible
This is an Ansible playbook configuration. This requires Ansible to be installed. We'll also update pip to prevent issues updating requirements.

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install -y ansible
    sudo pip3 install --upgrade pip

Make sure the Ansible requirements are installed next with the following command. This will install external roles & collections to the user's /home/<user>/.ansible/ dir. Some collections require additional requirement installations.

    ansible-galaxy collection install -r requirements.yml
    ansible-galaxy role install -r requirements.yml 
    pip3 install -r ~/.ansible/collections/ansible_collections/community/vmware/requirements.txt

Versions used in this pass:
 - ovirt.ovirt:1.6.6
 - ansible.posix:1.3.0
 - community.vmware:2.1.0
 - community.kubernetes:2.0.1
 - community.okd:2.1.0
 - kubernetes.core:2.2.3
 - community.windows:1.9.0

## Requirements - Ansible Vault
Ansible Vault is used to store credentials. We'll use a vault password file to encrypt & decrypt the Ansible Vault .yml file.

Create the vault password file with:

    vi ansible_vaultpasswd.user
    chmod 400 ansible_vaultpasswd.user
    echo 'ansible_vaultpasswd.user' >> .gitignore

One way to create encrypted password files is using stdin, i.e. typing it in a prompt. Press Ctrl-D twice to end input, if you use <Enter>, then the <Enter> is appended!

    echo 'ansible_vault.yml' >> .gitignore
    ansible-vault encrypt_string --vault-id donisaurs@ansible_vaultpasswd.user --stdin-name 'vmware_sa_username' >> ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@ansible_vaultpasswd.user --stdin-name 'vmware_sa_password' >> ansible_vault.yml

Use the following Ansible snippet to load the password file:
```
--- 
- name: Check inputs 
  hosts: localhost 
  connection: local 
  gather_facts: false 

  tasks: 
  - name: Check if Red Hat password file exists 
    stat: 
      path: passwords/ansible_vault.yml 
    register: vault_logins 

  - name: Terminate if password file is missing 
    fail: msg="Password file is missing" 
    when: not vault_logins.stat.exists 

  - name: Parse password var input file 
    include_vars: 
      file: passwords/ansible_vault.yml 
```

## Requirements - Linux VMware template

To be able to deploy Linux VMs we have to verify that open-vm-tools is installed on our template VM (done by default on RHEL 8.5), as well as several other packages required by deployPkg:

RHEL 8.5

  mkdir /mnt/rhel-dvd && mount -t iso9660 -o ro /dev/cdrom /mnt/rhel-dvd
  
  # Add for RHEL 8:
  cat << EOF >> /etc/yum.repos.d/rhel-dvd.repo
  [rhel-dvd-BaseOS]
  name=Red Hat Enterprise Linux $releasever - $basearch - DVD BaseOS
  baseurl=file:///mnt/rhel-dvd/BaseOS
  enabled=1
  gpgcheck=0
  
  [rhel-dvd-AppStream]
  name=Red Hat Enterprise Linux $releasever - $basearch - DVD AppStream
  baseurl=file:///mnt/rhel-dvd/AppStream
  enabled=1
  gpgcheck=0
  EOF
  
  yum clean all
  yum --noplugins list

  yum install open-vm-tools perl cloud-init


## Execution

To execute a playbook and capture the output (--ask-become-pass not needed):

  ansible-playbook -i inventory.yml deploy_OKD_3.11_nodes.yml --vault-id donisaurs@passwords/ansible_vaultpasswd.user 2>&1 | tee -a output_run_20210507_1345.log




--- WIP from here on ---

To be able to manage hosts through Ansible Python needs to be installed, including some other pre-reqs

    CentOS Steam 8
    --------------
    sudo yum install libcurl-devel gcc

And we need access to the host, e.g.:

    [donisaurs@odette ~]$ ssh-copy-id -i .ssh/id_rsa.pub donisaurs@ovirt02
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
    The authenticity of host 'ovirt02 (192.168.0.42)' can't be established.
    ECDSA key fingerprint is SHA256:exbHlqx1zBWPKUu8gqBnD3J5zxQvl+lR/99chkAjPgM.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    donisaurs@ovirt02's password: 

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'donisaurs@ovirt02'"
    and check to make sure that only the key(s) you wanted were added.

    [donisaurs@odette ~]$ ssh donisaurs@ovirt02.localdomain
    The authenticity of host 'ovirt02.localdomain (192.168.0.42)' can't be established.
    ECDSA key fingerprint is SHA256:RgNvq9xFe7obQXGzWM2s3rJ4EaA67d4/NLqkIexRmn4.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added 'ovirt03.localdomain' (ECDSA) to the list of known hosts.
    Activate the web console with: systemctl enable --now cockpit.socket


