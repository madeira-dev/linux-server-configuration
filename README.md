# Linux Server Configuration

This is an implementation of the project explained here: https://roadmap.sh/projects/configuration-management

This project uses a Google Cloud Platform (GCP) Virtual Machine (VM) as the Linux Server.

Add the GCP Linux Server VM to the inventory.ini file:
```
[gcp_hosts]
e2micro-linux-server ansible_host=34.135.89.198 ansible_port=22 ansible_user=gabrielmadeira2002 ansible_ssh_private_key_file=~/.ssh/id_ansible ansible_connection=ssh ansible_become=true
```

One important thing about the inventory.ini file is that each line should be one single host. Otherwise, we get an error

Before running the setup.yml, we need to make sure we can ssh into the server as Ansible. We need to create an ssh keypair for ansible to access it.

Generating the key pair (in local machine/control machine:
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_ansible
```

Now we copy the "id_ansible.pub" file (public key) to the VM under .ssh/ directory and write it to .ssh/authorized_keys in the VM.

To make sure it is working, we can use this command:
```
ansible -i inventory.ini e2micro-linux-server -m ping
```
If we get a pong as response then it is working!

Before running the ansible playbook we need to make sure the linux server user has sudo permissions to run commands such as 'apt update'.

We have to log in the VM through GCP and add the following line to the end of the file 'visudo':
```
gabrielmadeira2002 ALL=(ALL) NOPASSWD: ALL
```

We can then finally run the playbook
```
ansible-playbook -i inventory.ini setup.yml --tags nginx
```
