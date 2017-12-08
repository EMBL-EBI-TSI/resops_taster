# Practical 2
## Configuring a new machine with ansible

1. First login to your demo machine using the guide here: . After logging in, check to see that terraform and ansible are installed with `terraform -v` and `ansible --version`. If you get command not found for any of these please ask your friendly course guide for help.

2. If the machine from you previous practical is still up, good, we will use it again in this practical. If not, run `terraform apply` in the directory containing your `instance.tf` file and wait for it to complete. Make a note of your instance's IP address (as presented by `terraform show` under the `network_interface.0.access_config.0.nat_ip` variable).

3. In practical 1, we installed an nginx server by ssh'ing into our machine and running the install command manually. This does not scale well of course so in this practical we will use ansible to do the same thing. First, from your home directory (`cd ~`) make a directory called 'practical2' (`mkdir practical2`) and `cd` into it.

4. In this directory, create a file called `nginx.yaml` and open it with your favourite editor. Ansible uses yaml syntax, which provides a good balance between human and machine readability. Put the following contents in the file:
```yaml
---
# Define the name of the host we want to configure
hosts: webserver
tasks:
  - name: Install nginx
    package:
      name: nginx
      state: latest
```
We use the 'package' module to install nginx. There are many other modules, such as 'file' to create, edit and delete files, 'command' to execute shell commands and many more. For an overview of modules we can use, see: https://docs.ansible.com/ansible/latest/list_of_all_modules.html

5. Next we will need to tell Ansible where to find our host using an inventory file. Open a file called `inventory.ini` and put in the following contents:
```
[webserver]
x.x.x.x <- replace this line with your terraform provisioned ip from step 2
```
Remember to save and close the file.

6. We will now run ansible with our created configuration: `ansible-playbook -i inventory.ini nginx.yaml`
