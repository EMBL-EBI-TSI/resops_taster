GitHub repository hosting the practicals code: https://github.com/EMBL-EBI-TSI/resops_taster

# Practical 1
## Deploying a virtual machine with Terraform

1. Log into your instance using the guide here: https://goo.gl/Yx27bK

2. Your environment has been set up so that Terraform is already installed. Test this by running `terraform -v`. If you get ‘command not found’, contact your friendly course guides for help before continuing.

3. Make sure you are in your home directory (`cd ~`)

4. We’ll create a basic terraform file that boots a single server and allows us to SSH in. Create a file called ‘instance.tf’ and open it in your favorite editor. (You may need to install it, e.g. `sudo yum install vim/nano/emacs`)

5. We’ll define the Google provider first with some basic front matter:

```HCL
provider "google" {
  credentials = "${file("resops.json")}"
  project = "resops-taster"
  region = "europe-west1-b"
}

```
6. Next we'll define our basic vm instance:
```HCL
resource "google_compute_instance" "default" {
  # Change this to something interesting to prevent conflicts
  name         = "testmachine"
  
  machine_type = "n1-standard-1"
  zone         = "europe-west1-b"
  # This bit will fail! Change the tag because we need it to be unique later on. Only dashes and alphanumerics are allowed.
  tags         = ["change_this"]

  boot_disk {
    initialize_params {
      # We use Centos here, but this could be Ubuntu, CoreOS or anything really
      image = "projects/centos-cloud/global/images/centos-7-v20171129"
    }
  }

  # We can leave this empty to get a 10GB SSD disk, which suits our needs today
  scratch_disk {
  }

  network_interface {
    # We need to define a subnetwork, so we use the default of the project
    subnetwork = "projects/resops-taster/regions/europe-west1/subnetworks/default"
    access_config {
      # We leave the external ip empty, so it will get auto-assigned
      nat_ip = ""
    }
  }  
}
```

7. Save the file and exit. Then run `terraform init` to initalize the Google Cloud plugin for terraform, then run `terraform apply`. Some basic output will scroll by and finally your machine will have been created.

8. So now we can connect to it with ssh. Find the ip of your new machine using `terraform show` and find the value for `network_interface.0.access_config.0.nat_ip`. Then ssh to it: `ssh xx.xx.xx.xx`. 

9. You should now be in your new machine, try and install for example an nginx server with `sudo yum install nginx` (optional, more on this later). Exit to your deployment vm (after you are done playing around) by typing ‘exit’ or pressing Ctrl-D.

10. To make our terraform setup easily customizable, we can use variables to change things on a per-deployment basis. Open the instance.tf file and change the ‘name’ line of your VM instance to be as follows: 

```HCL
name = "${var.name}-machine"
```

Then, at the end of your file, add the definition of this variable:

```HCL
variable "name" {
  type = "string"
}
```

Save and exit, and run ‘terraform apply’. You will be asked to enter a name, and your machine will be modified to include the new name. You can use variables for all sorts of things that would need to be changed on a per-deployment basis, such as ip-addresses, passwords, file inputs, etc. 

11. Lastly, we need to tear down our infrastructure again. Run ‘terraform destroy’. Type ‘yes’ to confirm. You will be asked for the name variable again, but you don't have to enter anything. Terraform will now destroy your VM. Please don’t leave your machine running, so we can save resources as much as possible.

This concludes the first practical.
