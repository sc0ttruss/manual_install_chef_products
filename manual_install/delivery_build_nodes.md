# Manually installing Delivery Build Nodes
This guide explains how to install Delivery build nodes by hand.

### Getting and installing the packages
Obtain and install the following packages using the instructions for [Getting and Installing Packages](./getting_packages.md)

* chefdk
* delivery-cli
* opscode-push-jobs-client

The following 3rd party packages are also required
* git (Should be in the OS's repo)
* runit (Can be obtained from https://packagecloud.io/imeyer/runit)

You will also need to install some Ruby gems which can be obtained from https://rubygems.org

    chef gem install knife-supermarket
    chef gem install knife-push

    # Optional for Sentry Raven
    # chef gem install sentry-raven

If you don't have internet access or a local gem server you will need to download them (and all their dependencies) manually and copy them over.

# Configuring the build node

### Create a Push Jobs configuration
Create ```/etc/chef/push-jobs-client.rb``` with contents similar to the following

    LC_ALL='en_US.UTF-8'

    # Chef server connect options
    chef_server_url   'https://chef.myorg.chefdemo.net/organizations/myorg'
    node_name         'builder1.myorg.chefdemo.net'
    client_key        '/etc/chef/client.pem'
    trusted_certs_dir '/etc/chef/trusted_certs'
    verify_api_cert   true
    ssl_verify_mode   :verify_peer

    whitelist({
	    'chef-client'  => 'chef-client',
	    'delivery_cmd' => 'delivery_cmd',
	    'delivery-cli' => 'delivery-cli'
    })

    # We're under runit, so don't output timestamp
    Mixlib::Log::Formatter.show_time = true

### Create a runit configuration to start Push Jobs daemon
Create the required directories

    mkdir -p /etc/sv/opscode-push-jobs-client/log/main
    mkdir -p /etc/sv/opscode-push-jobs-client/env
    mkdir -p /etc/sv/opscode-push-jobs-client/control
    mkdir -p /var/log/opscode-push-jobs-client

    touch /etc/sv/opscode-push-jobs-client/log/config
    ln -s /var/log/opscode-push-jobs-client/config /etc/sv/opscode-push-jobs-client/log/config

Create a script ```/etc/sv/opscode-push-jobs-client/run``` with 755 permissions

    #!/bin/sh
    exec 2>&1
    exec /opt/push-jobs-client/bin/pushy-client -l info  -c /etc/chef/push-jobs-client.rb

Create a script ```/etc/sv/opscode-push-jobs-client/log/run``` with 755 permissions

    #!/bin/sh
    exec svlogd -tt /var/log/opscode-push-jobs-client

Create a symlink in the init.d directory to sv

    ln -s /etc/init.d/opscode-push-jobs-client /sbin/sv

Create a symlink for pushy in the service directory

    ln -s /etc/sv/opscode-push-jobs-client /etc/service/opscode-push-jobs-client

### Create the dbuild user and give it some credentials


### Set permissions so dbuild can read cheffy stuff
