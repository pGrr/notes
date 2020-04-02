# Laravel 7.x

# Server requirements

* The Laravel framework has a few system requirements
  * All of these requirements are satisfied by the Laravel Homestead virtual machine ([see more](https://laravel.com/docs/7.x/homestead))
  * Else, check all requirements are satisfied ([see more](https://laravel.com/docs/7.x#server-requirements))

# Laravel Homestead VM

* Laravel Homestead is an official, pre-packaged Vagrant box that provides all Laravel requirements ([see more](https://www.vagrantup.com/) about Vagrant)
  * includes Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, and all other Laravel requirements ([see all](https://laravel.com/docs/7.x/homestead#included-software))
  * Requires VirtualBox (or VMWare) and Vagrant
  * If you are using Windows, you may need to enable hardware virtualization (VT-x) 
* Once VirtualBox / VMware and Vagrant have been installed:
  * `vagrant box add laravel/homestead`
  * Set the `provider` key in your `Homestead.yaml`
    * e.g. `provider: virtualbox`
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
  * The `sites` property allows you to easily map a "domain" to a folder on your Homestead environment. Again, you may add as many sites to your Homestead environment as necessary
    ```yaml
    sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    ```
    * If you change the sites property after provisioning the Homestead box, you should re-run `vagrant reload --provision` to update
  * if you are experiencing issues while provisioning you should destroy and rebuild the machine via `vagrant destroy && vagrant up`
  * if you install Homestead "per-project" you should check [automatic host resolution](https://laravel.com/docs/7.x/homestead#hostname-resolution)