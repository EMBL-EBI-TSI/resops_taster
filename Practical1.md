GitHub repository hosting the practicals code: https://github.com/EMBL-EBI-TSI/resops_taster

# Practical 1
## Deploying a virtual machine with Terraform

1. Log into your instance using the guide here: 

2. Your environment has been set up so that Terraform and Ansible are already installed. Test this by running `terraform -v’ and ‘ansible --version`. If you get ‘command not found’ for either of these, contact your friendly course guides for help before continuing.

3. Create a folder called ‘practical1’ (‘mkdir practical1’) and cd into it.

4. We’ll create a basic terraform file that boots a single server and allows us to SSH in. Create a file called ‘instance.tf’ and open it in your favorite editor. (You may need to install it, e.g. ‘sudo yum install vim/nano/emacs’)

5. We’ll define a very basic VM first. Put the following contents in the file:

```
# Create a web server
resource "openstack_compute_instance_v2" "basic" {
  # Change this to something better!
  name            = "sometest_machine"
  # This is the id of a pre baked image in openstack
  image_id        = "3e8781ee-acfd-4f10-9884-5471378792e7"
  # This determines the size of the VM
  flavor_name     = "s1.tiny"
}
```

9. Save the file and exit. Then run ‘terraform apply’. Some basic output will scroll by and finally your machine will have been created. Login to the Google Cloud Console (console.cloud.google.com) to see for yourself; Click the 'hamburger' menu top left -> Compute Engine -> VM instances.

11. So now we can connect to it with ssh. Find the ip of your new machine using ‘terraform show’ and find the value for ‘network.0.fixed_ip_v4’. Then ssh to it with your key: ‘ssh xx.xx.xx.xx’. Are you able to connect?

12. The new machine is refusing our connection because of its default firewall. So we’ll need to create a new rule and assign it. Open your instance.tf file, and add the following at the top: 

```
# Create a security group
resource "openstack_compute_secgroup_v2" "demo_secgroup" {
  # Again, remember to change this
  name = "somename_secgroup"
  description = "basic demo secgroup"

  rule {
    from_port = 22
    to_port = 22
    ip_protocol = "tcp"
    cidr = "0.0.0.0/0"
  }
}
```

Next go into the machine definition and add 

security_groups = ["${openstack_compute_secgroup_v2.demo_secgroup.name}"]

Right below the key_pair entry we added in the previous step. Save and exit.

13. With your new file, run terraform apply again. You will see a new security group being created and your machine will be modified to include the new security group. Try to ssh in again. Is it working?

14. You should now be in your new machine, try and install for example an nginx server with ‘sudo yum install epel-release && sudo yum install nginx’ (optional). Exit to your deployment vm (after you are done playing around) by typing ‘exit’ or pressing Ctrl-D.

15. To make our terraform setup easily customizable, we can use variables to change things on a per-deployment basis. Open the instance.tf file and change the ‘name’ line of your VM instance to be as follows: 

```
name = "${var.name}_machine"

Then, at the end of your file, add the definition of this variable:

variable "name" {
  type = "string"
}
```

Save and exit, and run ‘terraform apply’. You will be asked to enter a name, and your machine will be modified to include the new name. You can use variables for all sorts of things that would need to be changed on a per-deployment basis, such as ip-addresses, passwords, file inputs, etc. We can use it anywhere we want to, so we could use it to name the security group and keypair too in our example. Try using the ‘name’ variable to name these two resources by editing your instance.tf file and redeploy your infrastructure. Did it work? (check the Horizon interface!)

16. Lastly, we need to tear down our infrastructure again. Run ‘terraform destroy’
Type ‘yes’ to confirm. Terraform will now destroy your VM. Please don’t leave your machine running, so we can save resources as much as possible.

This concludes the first practical.
