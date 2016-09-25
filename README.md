# Vagrant-Ansible-PHP7 #

Ansible provisioning of the Vagrant box for multiple PHP7 projects. It is designed so that you don't need to have things like webserver or Composer installed on your local system. You'll be able to do everything from within the virtual box.

That setup is heavily inspired by [https://github.com/hashbangcode/vlad](https://github.com/hashbangcode/vlad) and [Laravel Homestead](http://laravel.com/docs/4.2/homestead).

## What's inside ##

* Ubuntu 16.04 64-bit
* Nginx
* PHP7-FPM
* MySQL 5.7 + Adminer
* Codeception + PhantomJS + Selenium + Chrome
* NodeJS + Grunt + Bower + Gulp
* Composer
* Drush
* Git, Vim, Midnight Commander, etc.

## Supported types of projects ##

* Drupal 8 (not tested for Drupal 7)
* Laravel
* Custom PHP projects with a virtual host + customizable path to index.php
* Custom PHP projects without a virtual host (quite handy for code katas).

You can also specify if you would like the database to be created.

## Prerequisites ##

1. Linux or Mac
2. Vagrant ([latest version](https://www.vagrantup.com/downloads.html))
3. Ansible ([installation instructions](http://docs.ansible.com/intro_installation.html))
4. Virtual Box ([download](https://www.virtualbox.org/wiki/Downloads))
5. NFS (This comes pre-installed on Mac OS X 10.5+ (Leopard and higher))

    `sudo apt-get install nfs-kernel-server nfs-common portmap`

6. vagrant-triggers plugin

    `vagrant plugin install vagrant-triggers`

## Setup steps ##

1. Copy `example.settings.yml` to `settings.yml`

    `cp example.settings.yml settings.yml`

2. Open `settings.yml` with your favorite editor.

  * Provide your details for git setup.
  * Pick a webserver hostname. Phpinfo will be available at this URL.
  * Setup virtual hosts. See **Virtual hosts** section
  * Change the box IP address (if needed).
  * Change to box name to your liking.
  * Set the amount of RAM for the box (MB).
  * Change any other settings, though default are usually sufficient.

3. Run `vagrant up`. You will be asked for your system password in the beginning and in the end of the installation.
4. Run `vagrant ssh` to login to your box. All sites folders are in `~/sites` derectory.

## Virtual hosts ##

You can have as many sites as you want on a single box.

    # Virtual hosts
    - alias: domain.local
      path: /absolute/path/to/project/folder/on/your/local/machine
      # Set the type of the virtual host
      # This can be one of "none", "default", "drupal", "laravel"
      vhost_type: "default"
      custom_aliases: sub1.domain.local sub2.domain.local
      # If you don't want the database to be created, you can simply remove the next line.
      db: projectdb
      # If you use the "default" vhost type, you can specify the folder where your index.php is located.
      # Example: "", "/public", "/www". Please use the slash, when the path is not empty. NO TRAILING SLASH.
      index_file_folder: ""

Your project folders must exist but shouldn't contain any code. You will create a new application or clone the existing one from within the virtual box.

Virtual hosts are updated every time you run `vagrant up` or `vagrant reload`.

## Creating an application ##

Assuming that you've already created a virtual host for your app `domain.local`, go to `~/sites/domain.local`.

Your application must reside in that folder, no subfolders.

    ~/sites/domain.local/index.php (For Drupal projects)
    ~/sites/domain.local/public/index.php (For Laravel projects)

Open your browser and go to `domain.local/`. Job done.