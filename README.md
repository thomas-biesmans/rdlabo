# rdlabo

This is an Ansible playbook configuration. This requires Ansible to be installed.

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install -y ansible

Make sure the Ansible requirements are installed next with the following command. This will install external roles & collections to the user's /home/<user>/.ansible/ dir:

    ansible-galaxy collection install -r requirements.yml
    ansible-galaxy role install -r requirements.yml 

Versions used in this pass:
 - ovirt.ovirt:1.6.6
 - ansible.posix:1.3.0
 - community.vmware:2.1.0
 - community.kubernetes:2.0.1
 - community.okd:2.1.0
 - kubernetes.core:2.2.3

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


To execute a playbook and capture the output:

    ansible-playbook -i inventory.yml ovirt-provision.yml --ask-become-pass --vault-id donisaurs@passwords/vaultpassword 2>&1 | tee -a output_run_20210507_1345.log