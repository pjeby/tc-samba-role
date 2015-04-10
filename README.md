tc-samba
========

Based on [Bipedu's Samba Server on TinyCode linux HOWTO](https://bipedu.wordpress.com/2014/05/27/samba-server-on-tinycore-linux-howto/), this role installs and configures a samba server on a TinyCore Linux (such as `boot2docker`), complete with home directory creation, Samba password configuration,  updates to `/opt/.filetool.lst`, etc.


Requirements
------------

A TinyCore Linux installation with a working `tce-load` command, that Ansible can ssh into.

If you want the installation to persist (so that you don't have to run this role after every reboot), you will need a persistent `/tce` directory, and you will need to arrange to run `/usr/local/etc/init.d/samba start` at bootup.

If you're using `docker-machine` or `boot2docker`, you may not be able to arrange a persistent `tce` directory, in which case you should use the [boot2docker-persist](https://github.com/pjeby/boot2docker-persist) role first, to simulate standard TinyCore extension persistence and backups.  You will then need to set the `startup_script` variable for this role to `/var/lib/boot2docker/bootlocal.sh`

Speaking of backups, this role does not back up the configuration it creates, even though it will ensure that the `smb.conf` and the password database are included in `/opt/.filetool.lst`.  You should manually run the `backup` command once your system is working the way you want it.

Finally, note that this role assumes you are using it to do ALL Samba configuration tasks; it will **silently destroy any existing Samba configuration, inlcuding passwords**.  You have been warned.


Role Variables
--------------

* `samba_users` - a dictionary mapping user names to passwords -- the default is a `tc` user with a password of `tc`; if you're using a docker machine, you probably want to make a `docker` user instead.

* `samba_config` - a dictionary of options to be added to the `[global]` section of `/usr/local/etc/samba/smb.conf`.  Keys are option names, values are their values.

* `samba_shares` - a dictionary describing any shares to be created in `smb.conf`.  Keys are share names, values are dictionaries of options.  The default just sets up the standard home directory shares.

* `startup_script` - a shell script that the

Note that in both `samba_config` and `samba_shares`, you must enclose any `yes` or `no` values in quotes, so that they will be written to `smb.conf` as strings instead of boolean values.


Example Playbook
----------------

This example shows values identical to the default settings, but with `samba_users` and `startup_script` modified to show `boot2docker` usage:

    - hosts: my-docker-vm
      roles:
        - role: pjeby.tc-samba

          samba_users:
            docker: mysmbpassword

          startup_script: /var/lib/boot2docker/bootlocal.sh

          samba_config:
            workgroup: Workgroup
            "server string": Samba Server
            security: user
            "log file": /usr/local/samba/var/log.%m
            "max log size": 50
            browsable: "yes"
            "server signing": auto

          samba_shares:
            homes:
              comment: Home Directories
              browsable: "no"
              writable: "yes"


License
-------

MIT
