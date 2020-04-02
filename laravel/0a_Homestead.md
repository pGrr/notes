# Laravel Homestead VM

* Laravel Homestead is an official, pre-packaged Vagrant box that provides all Laravel requirements ([see more](https://www.vagrantup.com/) about Vagrant)
  * includes Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, and all other Laravel requirements ([see all](https://laravel.com/docs/7.x/homestead#included-software))
  * Requires VirtualBox (or VMWare) and Vagrant
  * If you are using Windows, you may need to enable hardware virtualization (VT-x) 

# Install

Once VirtualBox / VMware and Vagrant have been installed:

* Global install
  * shares the same Homestead box across all of your projects
  * `vagrant box add laravel/homestead`
* Per project install
  * Configure a local Homestead instance
  * May be beneficial if you wish to ship a `Vagrantfile` with your project, allowing others working on the project to `vagrant up`
  * `composer require laravel/homestead --dev`
  * `php vendor/bin/homestead make`
    * generates the `Vagrantfile` and `Homestead.yaml` file in your project root. The `make` command will automatically configure the sites and folders directives in the `Homestead.yaml` file
    * `vagrant up`
    * access your project at `http://homestead.test` in your browser
  * Remember, if you are not using automatic hostname resolution ([see more](https://laravel.com/docs/7.x/homestead#hostname-resolution)) you will still need to add an `/etc/hosts` file entry for `homestead.test` or the domain of your choice.


# Configure

## Set Vagrant provider (VirtualBox or VMWare)

  * Set the `provider` key in your `Homestead.yaml`
    * e.g. `provider: virtualbox`

## Synched folders

* The `folders` property of the `Homestead.yaml` file lists all of the folders you wish to share with your Homestead environment. They will be kept in sync between your local machine and the Homestead environment
* You can list there as many folder as you want (best practice is to include projects root folder, never use relative paths)
    ```yaml
    folders:
    - map: ~/code/project1
        to: /home/vagrant/project1
    ```
* All vagrant options can be included:
* e.g. `type: "nfs"` to use NFS sync folders
* e.g.
    ```yaml
    folders:
    - map: ~/code/project1
        to: /home/vagrant/project1
        type: "rsync"
        options:
            rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
            rsync__exclude: ["node_modules"]
    ```

## Sites

* The `sites` property allows you to easily map a "domain" to a folder on your Homestead environment. Again, you may add as many sites to your Homestead environment as necessary
    ```yaml
    sites:
    - map: homestead.test
        to: /home/vagrant/project1/public
    ```
* If you change the sites property after provisioning the Homestead box, you should re-run `vagrant reload --provision` to update
* if you are experiencing issues while provisioning you should destroy and rebuild the machine via `vagrant destroy && vagrant up`
* if you install Homestead "per-project" you should check [automatic host resolution](https://laravel.com/docs/7.x/homestead#hostname-resolution)

## Optional features

* Many optional extensions are available by simply setting a flag into the `Homestead.yaml` configuration file ([see more](https://laravel.com/docs/7.x/homestead#installing-optional-features))
  * e.g. `mysql8`, `python`, `couchdb`, `docker`, `mariadb`, `mongodb`, `neo4j`, `golang`, `blackfire`, `webdriver`, `rabbitmq`, etc

## Defining aliases for any command

* You may add Bash aliases to your Homestead machine by modifying the aliases file within your Homestead directory, e.g.:
    ```bash
    alias c='clear'
    alias ..='cd ..'
    ```
* `vagrant reload --provision` - necessary to make them available

# Launching / destroying

* `vagrant up` 
  * Global install: from your Homestead directory (should be `~/Homestead`) will boot the virtual machine (and automatically configure your shared folders and Nginx sites)
  * Per project install: from the project root folder
* `vagrant destroy --force` to destroy the machine

# Connecting via SSH

* `vagrant ssh` - from your Homestead directory.

# Connecting to database
