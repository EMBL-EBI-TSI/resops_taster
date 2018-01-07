# Practical 2
## Configuring a new machine with Ansible

1. First login to your demo machine using the guide here: . After logging in, check to see that terraform and ansible are installed with `terraform -v` and `ansible --version`. If you get `command not found` for either of these please ask your friendly course guide for help.

2. If the machine from your previous practical is still up, we will use it again in this practical. If not, run `terraform apply` in the directory containing your `instance.tf` file and wait for it to complete. Make a note of your instance's IP address (as presented by `terraform show` under the `network_interface.0.access_config.0.nat_ip` variable).

3. In practical 1, we installed an nginx server by ssh'ing into our machine and running the install command manually. This does not scale well of course so in this practical we will use Ansible to do the same thing. Please make sure you are again working from your home directory for this practical (`cd ~`).

4. Create a file called `nginx.yaml` and open it with your favourite editor. Ansible uses yaml syntax, which provides a good balance between human and machine readability. Put the following contents in the file:
```yaml
---
# Define the name of the host we want to configure
- hosts: webserver
# Become means we will become root, we need this to install packages
  become: yes
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: latest
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes

```
We use the 'package' module to install nginx and the 'service' module to start it. There are many other modules, such as 'file' to create, edit and delete files, 'command' to execute shell commands and many more. For an overview of modules we can use, see: https://docs.ansible.com/ansible/latest/list_of_all_modules.html

5. Next we will need to tell Ansible where to find our host using an inventory file. Open a file called `inventory.ini` and put in the following contents:
```
[webserver]
x.x.x.x <- replace this line with your terraform provisioned ip from step 2
```
Remember to save and close the file.

6. We will now run ansible with our created configuration: `ansible-playbook -i inventory.ini -e 'host_key_checking=False' nginx.yaml`

7. When the command has completed successfully, our machine is now running a webserver with the default welcome page. Try visiting the IP address of your machine in your browser. Is it working?

8. You won't be able to get any web traffic from the machine, because Google Cloud's default firewall is preventing access over port 80. Let's go back to our Terraform and add an exception. Open your `instance.tf` file for editing again and add the following at the top:
```yaml
resource "google_compute_firewall" "default" {
  name          = "webserver-firewall"
  network       = "projects/resops-taster/regions/europe-west1/networks/default"
  source_ranges = ["0.0.0.0/0"]

  # Again, this will break. Change the tag to the tag you have defined for your instance lower down in this file
  target_tags   = ["change_this"]

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }
}
```
Note that the ordering of entries in the Terraform file does not matter. We describe a desired state, not a sequence of commands.

9. Now that you've updated your Terraform definition, run `terraform apply` again and type 'yes' when prompted. Note that Terraform leaves your instance in place and merely adds a firewall rule to the current setup. This is a powerful feature when making iterative changes to large sets of infrastructure.

10. With the new firewall rule in place, type your created instance's IP into your browser's address bar. This should give you the nginx welcome page. Congratulations, you've deployed a webserver without ever ssh'ing into it. You are now ready to start working at scale in the cloud ;)

11. Please remember to run `terraform destroy` and confirm when promted to save resources where possible. Thanks.

This concludes practical 2.
