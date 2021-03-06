# Outpost
Portable WordPress development environments with Vagrant, VMware, and WP-CLI.

http://outpost.rocks

**Version**: 0.1.7  
**Licence**: MIT  
**Author**: Nick Cernis [@nickcernis](http://twitter.com/nickcernis)  

## Beta warning
**Outpost is experimental and in flux.** It currently only supports VMware virtual machines, but may support VirtualBox in the future. Tested on Mac. Untested on Windows and Linux (should work, but do file bug reports!). Contributions and feature requests are welcome.

## WordPress development environments on demand
Outpost creates a headless virtual server running Ubuntu 14.04 inside your local machine pre-provisioned with Apache 2.4.7, MySQL 5.5 and PHP 5.5 (using a [custom box](https://github.com/nickcernis/outpost-packer)). Then it downloads the latest WordPress and runs the WordPress install for you.

### Use Outpost to:

1. Launch a fresh WordPress development environment with one command. Like MAMP/WAMP, but without any configuration, setup wizards, or a GUI to get in your way.  (See "Getting Started".)
2. Clone a remote site to work on locally in a similar environment. (Experimental. See "Cloning".)
2. Package a copy of an existing theme, plugin, or entire WordPress site to distribute to a development team, who can recreate your environment and see the working website with one command. (See "Packaging for distribution".)

### You end up with:

- A fresh copy of WordPress running from the virtual machine, accessible in your browser at [http://my.outpost.rocks](http://my.outpost.rocks) with WP username 'outpost' and password 'outpost'.
- A 'wp/wp-content' folder in your project's directory, synced live with the virtual machine. Just drop your themes and plugins in, then get coding. This lets you develop with any IDE or editor on your system to get the benefits of using a virtual machine – portability, uniformity – without having to see it or interact with it via the command line (unless you want to).

## Getting started

Before you use Outpost, you'll need to:

1. Purchase and install [VMware Fusion](http://www.vmware.com/products/fusion/) (Mac) or [VMware Workstation](http://www.vmware.com/products/workstation/) (Windows/Linux).
2. [Install Vagrant.](http://docs.vagrantup.com/v2/installation/)
3. Purchase and install the [Vagrant VMware plugin](http://www.vagrantup.com/vmware).

Note: By default, vagrant uses the VirtualBox provider and will look for VirtualBox machines when you run `vagrant up`. You can tell it to use VMware boxes instead with `vagrant up --provider=vmware_fusion`. Since typing all that is a pain, you might like to tell vagrant to use VMware by default instead of VirtualBox by setting the [default provider](http://docs.vagrantup.com/v2/providers/default.html). That way, you can just type `vagrant up` and launch the VMware box by default.

## 1. Creating a fresh development environment

You can launch a new development environment with Outpost like this:

1. [Download Outpost.](http://outpost.rocks)
2. Rename your `outpost` directory to a project name of your choosing.
3. Change to that directory and type `vagrant up` in the terminal.

The first time you do this, it takes a while – Vagrant has to download the Outpost disk image from vagrantcloud.com (about 550MB). Subsequent `vagrant up`s are faster because Vagrant caches the outpost disk image on your hard drive.

Once it's done, visit http://my.outpost.rocks in your browser to see your new development site. The latest WordPress core is already installed and configured with admin username "outpost" and password "outpost". Vagrant will sync any files you drop or edit in `wp/wp-content` to the virtual server.

## 2. Cloning a live site for local development (experimental)
Outpost is also a WordPress plugin – install it and you'll find that it can download your site as an Outpost that you can launch on your local machine with `vagrant up`. The site packaging and download process is buggy on some servers, though, so it's currently only for the brave. (I'll post it to the WordPress plugin repository when it's stable.)

If you need to recreate a live site locally using Outpost, at the moment I recommend that you:

1. [Download Outpost.](http://outpost.rocks)
2. Rename the `outpost` directory with your project name.
3. Download the contents of your remote `/wp-content/` folder to Outpost's `/wp/wp-content` folder.
4. Dump your live MySQL database (with PHPMyAdmin or with [WP Migrate DB](https://wordpress.org/plugins/wp-migrate-db/)), rename it “outpost.sql” and put it in `/wp/wp-data`
5. Run `vagrant up`.

Eventually Outpost will automate steps one to four so that you can click a button to package and download your live site and then run `vagrant up` locally.

## 3. Packaging for distribution

This feature is planned – you'll be able to share sites with other developers or package a development site or live site to install on a production or staging server.

## New to Vagrant and virtual machines? Read this:

If you're not familiar with [Vagrant](http://vagrantup.com) or virtual machines, what you have after you type `vagrant up` is:

- A working copy of WordPress core running in VMware Fusion/Workstation on a virtual machine using Ubuntu 14.04. The machine's accessible in your browser at http://my.outpost.rocks, and via SSH using `vagrant ssh` from your project's root directory.
- A `wp/wp-content` folder on your local machine that syncs to Apache's `/var/www/html/wp-content` folder on the virtual machine. This lets you build themes and plugins locally with your favourite editor – changes you make to files in `wp/wp-content` automatically sync to the virtual machine.

How that happened is:

- [VMware](https://www.virtualbox.org/wiki/Virtualization) runs a [preconfigured server](https://github.com/nickcernis/outpost-packer) on your machine, which saves you having to mess around and install MySQL and other WordPress dependencies.
- Vagrant launches the server, syncs folders between your machine and the server, and installs WordPress when you first run `vagrant up`.
- The A record for the my.outpost.rocks domain points to the IP address 192.168.53.53, which is the address of the VMware box on your local machine. When you visit that URL, your browser serves the website hosted in the virtual machine on your own computer. (The URL is *not* publicly accessible to your clients.)

## The workflow
When you're done working with Outpost or want to switch projects, you can either:

1. Suspend the virtual machine with `vagrant suspend`. This writes RAM to disk and pauses the machine, saving RAM and CPU cycles. To resume, type `vagrant resume`.
2. Destroy the virtual machine with `vagrant destroy -f`. This destroys the virtual machine, saving RAM, CPU cycles, and disk space, but resuming will take longer. To resume, type `vagrant up`. Outpost will recreate your WordPress site from the files in your `wp/wp-content` folder and from the database dump it creates every three minutes.

If you restart your computer without using `vagrant suspend` or `vagrant destroy -f`, you can bring your Outpost back again with `vagrant up` and/or `vagrant provision`.

Because all Outposts use the http://my.outpost.rocks URL and the same IP address, you should only ever run one Outpost at a time. You can develop multiple themes and plugins using one Outpost, though. Or you can suspend one Outpost and run another. Use the `vagrant global-status` command to see all of the Outposts you've ever created, and destroy ones you're not using to save disk space (with `vagrant destroy [box-id]`).

## How MySQL changes work
When you're running Outpost, it dumps the database every three minutes to /wp/wp-data/outpost.sql. When you run `vagrant up`, Outpost looks for the `outpost.sql` file and uses it to recreate the database. This means you can destroy the virtual machine and run `vagrant up` without losing all of your data.

This zero-configuration setup offers the potential to lose data, but I felt it was better than asking users to manually trigger backups. There may be better ways to saving state, though, such as using Vagrant's hooks to trigger SQL dumps after a suspend, halt, or destroy. For now, periodic SQL dumps seems adequate for general theme and plugin development, where database content is typically static and it's mostly theme and plugin files that are changing.

If you need to access MySQL from the console, the MySQL root password is 'mysql'. (You can SSH into the virtual machine using `vagrant ssh` from the command line. No user or password is needed.)

Outpost also ships with phpMyAdmin, accessible at http://my.outpost.rocks/phpmyadmin/ with username `root` and password `mysql`.

## Deployment and data syncing

Outpost is *not* yet designed to push theme, plugin, or database changes you've made back to a staging or production server. It's worth setting up a separate deployment strategy for that, perhaps using a service such as [Deploy](https://www.deployhq.com/).

I hope to make it easier to pull remote database changes back to your Outpost, but [WP Migrate DB Pro](https://deliciousbrains.com/wp-migrate-db-pro/) has you covered for now. Outpost overrides the site URL in wp-config.php, so you don't need to modify that, but you might consider using the [Root Relative URLs](https://wordpress.org/plugins/root-relative-urls/) plugin on the server and development box to avoid problems with full URLs that appear in your posts and pages.

## How Outpost is different

Outpost differs from traditional WordPress development environment setups such as WAMP, MAMP, and DesktopServer in these ways:

1. It uses Vagrant and VMware's virtualisation products to create a tiny server running on your own machine that better mimics the average shared server.
2. It automates the entire WordPress setup experience in one terminal command: `vagrant up`. That means less time configuring and more time coding.
3. It's easier to share your site with other developers – you can package it, zip it up, and send it to them.
4. Because it's a command line -driven workflow, it makes some developers happier (and drives others insane).
5. When you need it, you have complete control over the development server.

Outpost differs from other Vagrant-based WordPress developer setups such as [VVV](https://github.com/Varying-Vagrant-Vagrants/VVV) in a few ways:

1. Outpost uses a [custom box](https://github.com/nickcernis/outpost-packer) that includes PHP, MySQL, Apache, and WP-CLI, so that you don't have to watch all of those things build and install each time you run `vagrant up`. WordPress itself is the only thing that `vagrant up` adds during the provisioning process. (WP-CLI is also updated if needed.)
2. Outpost is a plugin – not just a Vagrant template. You can use it stand-alone to generate new WordPress environments, or you can install it on a WordPress blog and use it to generate an “Outpost” – a packaged group of files to download and use to recreate that site locally for development and debugging with `vagrant up`.
3. Outpost is optimised for theme and plugin development – it's not designed for hacking on WordPress itself. (It doesn't make WordPress core files available via vagrant's shared folder – it just shares `wp-content`.)

Outpost also differs from other projects in these ways:

1. Outpost is preconfigured with a live development URL: http://my.outpost.rocks. You don't have to create, remember, or maintain `example.dev` URLs, and you don't have to edit your `/etc/hosts` file to point made-up domains to an internal IP. (The A record for my.outpost.rocks already points to the Outpost virtual machine IP address.) This gives you one simple URL to hack on all your WordPress themes and plugins, with zero configuration.
2. Outpost has a handy website at http://outpost.rocks with short video walkthroughs (<em>coming soon!</em>) designed to make Outpost easier to adopt.

### TODO:

- Fix plugin issues when cloning large databases.
- Offer a way to choose system configuration options prior to startup. (To use nginx instead of Apache, for example.)

## Contributing

I welcome all contributions and feature requests. Please [file issues here](https://github.com/nickcernis/outpost-packer/issues). You can also [contact me on twitter](http://twitter.com/nickcernis/).

## Other notes

- If `wp/wp-content/` is blank, Outpost puts a clean copy of WordPress core's wp-content in there.
- If `wp/wp-content/` contains theme and plugin files, Outpost uses your wp-content directory to build your site.
- It's safe to delete the contents of the `wp/wp-content/` and `wp/wp-data` folders to reset your Outpost and force a new download of WordPress on the next `vagrant up`, but don't delete the directories themselves.
