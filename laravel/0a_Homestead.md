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
* You may add additional Nginx `fastcgi_param` values to your site via the `params` site directive ([See more](https://laravel.com/docs/7.x/homestead#site-parameters))

## Optional features

* Many optional extensions are available by simply setting a flag into the `Homestead.yaml` configuration file ([see more](https://laravel.com/docs/7.x/homestead#installing-optional-features))
  * e.g. `mysql8`, `python`, `couchdb`, `docker`, `mariadb`, `mongodb`, `neo4j`, `golang`, `blackfire`, `webdriver`, `rabbitmq`, etc

## Database backups

* Homestead can automatically backup your database when your Vagrant box is destroyed. To utilize this feature, you must be using Vagrant 2.1.0 or greater
* add the following line to your `Homestead.yaml` file:
  * `backup: true`
* Once configured, Homestead will export your databases to `mysql_backup` and `postgres_backup` directories when the `vagrant destroy` command is executed. These directories can be found in the folder where you cloned Homestead or in the root of your project if you are using the per project installation method.

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

# Database

## Connecting to database

* A `homestead` database is configured for both MySQL and PostgreSQL out of the box. To connect to your MySQL or PostgreSQL database from your host machine's database client, you should connect to `127.0.0.1` and port `33060` (MySQL) or `54320` (PostgreSQL). The username and password for both databases is `homestead` / `secret`.
* You should only use these non-standard ports when connecting to the databases from your host machine. You will use the default `3306` and `5432` ports in your Laravel database configuration file since Laravel is running within the virtual machine.

## Database snapshots

* Homestead supports freezing the state of MySQL and MariaDB databases and branching between them using Logical MySQL Manager. For example, imagine working on a site with a multi-gigabyte database. You can import the database and take a snapshot. After doing some work and creating some test content locally, you may quickly restore back to the original state. ([see more](https://laravel.com/docs/7.x/homestead#database-snapshots))

# Ports

By default, the following ports are forwarded to your Homestead environment:

* SSH: 2222 → Forwards To 22
* ngrok UI: 4040 → Forwards To 4040
* HTTP: 8000 → Forwards To 80
* HTTPS: 44300 → Forwards To 443
* MySQL: 33060 → Forwards To 3306
* PostgreSQL: 54320 → Forwards To 5432
* MongoDB: 27017 → Forwards To 27017
* Mailhog: 8025 → Forwards To 8025
* Minio: 9600 → Forwards To 9600

# Update

* `vagrant destroy`
* If you have installed Homestead via your project's `composer.json` file, you should 
  * ensure your `composer.json` file contains `"laravel/homestead": "^10"` and update your dependencies:
  * `composer update`
* `vagrant box update`
* `vagrant up`

## Forwarding Additional Ports

* If you wish, you may forward additional ports to the Vagrant box, as well as specify their protocol:
    ```yaml
    ports:
        - send: 50000
        to: 5000
        - send: 7777
        to: 777
        protocol: udp
    ```
# Use Homestead outside of Laravel

* Homestead supports several types of sites which allow you to easily run projects that are not based on Laravel. 
  * For example, we may easily add a Symfony application to Homestead using the symfony2 site type:
    ```yaml
    sites:
    - map: symfony2.test
      to: /home/vagrant/my-symfony-project/web
      type: "symfony2"
    ```
* The available site types are: `apache`, `apigility`, `expressive`, `laravel` (the default), `proxy`, `silverstripe`, `statamic`, `symfony2`, `symfony4`, and `zf`

# More info and additional features

* [See docs]() for more info or additional features, such as:
  * Debugging with XDebug
  * Profiling with BlackFire or XHGui
  * Environment variables
  * Adding fastcgi_param parameters
  * Configuring Cron Schedules
  * Use mail: Postfix mail transfer agent
  * Configuring Mailhog (easily catch your outgoing email and examine it without actually sending the mail to its recipients)
  * Configuring Minio: an open source object storage server with an Amazon S3 compatible API
  * Sharing Your Environment: Sometimes you may wish to share what you're currently working on with coworkers or a client
  * Supporting multiple PHP Versions
  * Flip between NGINX and Apache (for a `type: apache` site)
  * Configure Network Interfaces (e.g. DHCP, ip, bridge, etc)
  * Extend Homestead using the after.sh script 
  * Customize Homestead with `user-customizations.sh`
  * Virtualbox `natdnshostresolver` setting
  * Symbolic links on Windows specific settings

