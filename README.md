# RDLabo
## Requirements - Ansible
This is an Ansible playbook configuration. This requires Ansible to be installed. We'll also update pip to prevent issues updating requirements.

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install -y ansible
    sudo pip3 install --upgrade pip

Make sure the Ansible requirements are installed next with the following command. This will install external roles & collections to the user's /home/\<user\>/.ansible/ dir. Some collections require additional requirement installations.

    ansible-galaxy collection install -r requirements.yml
    ansible-galaxy role install -r requirements.yml

    # You can use a specific Python version by specifying python3.8:
    python3.8 -m pip install packaging --user
    python3.8 -m pip install -r ~/.ansible/collections/ansible_collections/community/vmware/requirements.txt --user
    python3.8 -m pip install -r ~/.ansible/collections/ansible_collections/kubernetes/core/requirements.txt --user
    python3.8 -m pip install pywinrm --user
    python3.8 -m pip install nsnitro --user
    python3.8 -m pip install jmespath --user
    python3.8 -m pip install ansible-pylibssh --user
    python3.8 -m pip install ttp --user

    # For Python 3.9
    python3.9 -m pip install packaging --user
    python3.9 -m pip install jinja2 --user
    python3.9 -m pip install resolvelib==0.8.1 --user
    python3.9 -m pip install -r ~/.ansible/collections/ansible_collections/community/vmware/requirements.txt --user
    python3.9 -m pip uninstall pyVmomi
    python3.9 -m pip install pyVmomi==7.0.3 --user
    python3.9 -m pip install -r ~/.ansible/collections/ansible_collections/kubernetes/core/requirements.txt --user
    python3.9 -m pip install pywinrm --user
    python3.9 -m pip install nsnitro --user
    python3.9 -m pip install jmespath --user
    python3.9 -m pip install ansible-pylibssh --user
    python3.9 -m pip install ttp --user
    

Versions used in the OKD pass:
 - ovirt.ovirt:1.6.6
 - ansible.posix:1.3.0
 - community.vmware:2.1.0
 - community.okd:2.1.0
 - kubernetes.core:2.2.3
 - community.windows:1.9.0
 - community.general:4.5.0

Versions used in the OpenShift 4 pass:
 - ovirt.ovirt:2.2.3
 - ansible.posix:1.4.0
 - community.vmware:2.9.1
 - community.kubernetes:2.0.1
 - community.okd:2.2.0
 - community.windows:1.11.0
 - community.general:5.6.0
 - kubernetes.core: 2.3.2

## Requirements - Ansible Vault
Ansible Vault is used to store credentials. We'll use a vault password file to encrypt & decrypt the Ansible Vault .yml file.

Create the vault password file with:

    mkdir passwords
    vi passwords/ansible_vaultpasswd.user
    chmod 400 passwords/ansible_vaultpasswd.user
    echo 'passwords/ansible_vaultpasswd.user' >> .gitignore

One way to create encrypted password files is using stdin, i.e. typing it in a prompt. Press Ctrl-D twice to end input, if you use \<Enter\>, then the \<Enter\> is appended!

    echo 'ansible_vault.yml' >> .gitignore
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'vmware_sa_username' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'vmware_sa_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'template_lin_username' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'template_lin_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'client_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'citrix_username' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'citrix_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'vmware_nsx_username' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'vmware_nsx_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'vmware_nsx_client_certificate_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'aruba_admin_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'fortigate_admin_password' >> passwords/ansible_vault.yml
    ansible-vault encrypt_string --vault-id donisaurs@passwords/ansible_vaultpasswd.user --stdin-name 'virtualconnect_admin_password' >> passwords/ansible_vault.yml

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

## Requirements - Template SSH certificate for Ansible deployment

There are two options two connect to a deployed VM:
  1. Using username and passwords if 'PasswordAuthentication yes' is set in /etc/ssh/sshd_config. Cleartext passwords aren't considered safe.
  2. Pre-deploying an 'ansible' user and public SSH key for the first connections. This can be used to deploy a new SSH key.

Create the SSH key with the following command:

  ssh-keygen -q -b 2048 -t rsa -N "" -f ~/.ssh/id_rsa_ansibledeployment

## Requirements - Linux VMware template

To be able to deploy Linux VMs we have to verify that open-vm-tools is installed on our template VM (done by default on RHEL 8.5), as well as several other packages required by deployPkg. We'll also create our 'ansible' user and import our SSH deployment certificate.

 ### VM Setting 

 We'll be using VMware as our default storageClass provider. This requires us to set the following advanced setting on our VMs. Adding it to the template is the easiest way:

  disk.enable

 ### RHEL 7.9
  yum update

  yum clean all
  yum --noplugins list
api.okd3.rdlabo.local:8443
  yum install open-vm-tools perl cloud-init yum-util

  useradd -d /home/ansible -m ansible -s /bin/bash
  passwd ansible
  usermod -a -G wheel ansible

  mkdir /home/ansible/.ssh
  chmod 700 /home/ansible/.ssh
  touch /home/ansible/.ssh/authorized_keys
  chmod 600 /home/ansible/.ssh/authorized_keys
  chown -R ansible:ansible /home/ansible/.ssh
  scp \<user\>@\<IP\>:/home/\<user\>/.ssh/id_rsa_ansibledeployment.pub /tmp/
  cat /tmp/id_rsa_ansibledeployment.pub >> /home/ansible/.ssh/authorized_keys
  rm /tmp/id_rsa_ansibledeployment.pub

 ### RHEL 8.5

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

  yum install open-vm-tools perl cloud-init yum-util

  useradd -d /home/ansible -m ansible -s /bin/bash
  passwd ansible
  usermod -a -G wheel ansible

  mkdir /home/ansible/.ssh
  chmod 700 /home/ansible/.ssh
  touch /home/ansible/.ssh/authorized_keys
  chmod 600 /home/ansible/.ssh/authorized_keys
  chown -R ansible:ansible /home/ansible/.ssh
  scp \<user\>@\<IP\>:/home/\<user\>/.ssh/id_rsa_ansibledeployment.pub /tmp/
  cat /tmp/id_rsa_ansibledeployment.pub >> /home/ansible/.ssh/authorized_keys
  rm /tmp/id_rsa_ansibledeployment.pub


## Execution

To execute a playbook and capture the output (--ask-become-pass not needed):

  ansible-playbook -i inventory.yml deploy_OKD_3.11_nodes.yml --vault-id donisaurs@passwords/ansible_vaultpasswd.user 2>&1 | tee -a output_run_20210507_1345.log

If you want to execute a specific tag, then you can do so as well:

  ansible-playbook -i inventory.yml deploy_OKD_3.11_nodes.yml --vault-id donisaurs@passwords/ansible_vaultpasswd.user --tags=storage_extend 2>&1 | tee -a output_run_20210507_1345.log








## Crap moved forward
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


