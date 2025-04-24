# Linux Server Configuration

This is an implementation of the project explained here: https://roadmap.sh/projects/configuration-management

The project description recommends using a cloud provider but for the purpose of learning more things this implementation uses Docker to start my server locally.

The Docker container for the linux server uses the jrei/systemd-ubuntu (systemd enabled) image.

To start the Docker container we can use the following command:

```
docker run --name linux_server --privileged -dit -p 2222:22 ubuntu
```

Details of the command:
```
-dit -> starts container in detached mode;

--name -> specify the name of the container

-p -> specify the port to be open
```

If the ubuntu image isn't already downloaded, the command above will automatically download it and start the container.

Make sure to log in to the created container to install ssh since it comes uninstalled. Yes it could be done through Docker compose but for this project we are focusing on Ansible

Add the Docker container to the inventory.ini file:
```
[docker_hosts]
linux_server ansible_host=127.0.0.1 ansible_port=2222 ansible_user=root ansible_connection=ssh
```
One important thing about the inventory.ini file is that each line should be one single host. Otherwise, we will get an error

Before running the setup.yml to start configuring the linux server, we need to make sure we can ssh into the server. We need to create an ssh keypair for ansible to access it.

Generating the key pair (in local machine/control machine:
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_ansible
ssh-agent bash && ssh-add ~/.ssh/id_ansible
```

The second command will start a bash session with ssh user, in this session:
```
ssh-copy-id -i ~/.ssh/id_ansible.pub -p 2222 root@127.0.0.1
```

In this bash session we will be required to enter the password for the root user (the default user) in the docker container so make sure to have already set one because it comes with no password by default and won't be able to authenticate through ssh. For this case we need to access the docker container.

Still related to enabling ssh into the linux server container, be sure to have enabled the option to ssh as root with these commands:
```
sed -i 's/^#\?PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin yes/'       /etc/ssh/sshd_config
service ssh restart
```
After this we can finally ssh into the server as root. Therefore, we can upload the ssh key for ansible to the container.

After starting the Docker container as the linux server, run the Ansible setup.yml with the command:
```
ansible-playbook -i inventory.ini setup.yml --tags nginx
```
