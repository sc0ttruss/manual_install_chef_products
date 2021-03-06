# Manually installing Delivery Server
This guide explains how to install Delivery server by hand.

### Getting and installing the packages
Obtain and install the ```delivery``` package using the instructions for [Getting and Installing Packages](./getting_packages.md)

### Getting and installing a license file
* Request a licence file in the Chef ```#delivery-license``` Slack channel
* Unzip the license Zip file into ```/var/opt/delivery/license```

### Create a basic Delivery Configuration

    mkdir /etc/delivery

Edit [/etc/delivery/delivery.rb](../reference/delivery.rb.md) to contain something similar to:-

    delivery_fqdn 'delivery.myorg.chefdemo.net'
    delivery['chef_username'] = 'delivery_myorg'
    delivery['chef_private_key'] = '/etc/delivery/delivery_myorg.pem'
    delivery['chef_server'] = 'https://chef.myorg.chefdemo.net/organizations/myorg'

### Create a delivery user in Chef Server
Log in to the Chef server and create a user with admin rights that Delivery can use to manipulate cookbooks.

In this example we will create a ```delivery_myorg``` user in the ```myorg``` organisation.

    # Create a Chef user for Delivery to use
    chef-server-ctl user-create delivery_myorg Delivery MyOrg delivery@email.fake random_password

    # Add delivery_myorg to myorg
    chef-server-ctl org-user-add myorg delivery_myorg

Copy/paste the returned private key to the Delivery server as ```/etc/delivery/delivery_myorg.pem```

Add the delivery user you created to the ```admins``` group in the Chef UI.

### Converge Delivery

    delivery-ctl reconfigure

# Creating the Delivery Enterprise/Org(s)

### Generate Enterprise SSH credentials
The Enterprise SSH credentials are used by build nodes to perform GIT manipulations. They are required to create an Enterprise.

    cd /etc/delivery
    ssh-keygen -f myorg_ssh_key

Two files will be generated containing the public and private keys. These keys will need to be copied over to the build nodes later.

### Create the Enterprise
Create the organisation and load the SSH public key into it

    delivery-ctl create-enterprise myorg --ssh-pub-key-file=/etc/delivery/myorg_ssh_key.pub

Collect the admin and builder passwords from the screen and keep them safe.
