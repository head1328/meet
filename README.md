# Setup

Example of an Ansible Jitsi setup running on a Hetzner root server instance I named `hqrs1`.

**Before you start:**\
Copy ./host_vars/hqrs1.example to ./host_vars/hqrs1 and configure all required fields. Perhaps you want to rename your server instance. for this, change `hqrs1` to anything you want: ./host_vars/hqrs1, ./hosts, .hqrs1.yml

*Notice:\
I called my server hqrs1, just change any files and replace this value for your own needs.*

## Requirements

* Any domain that points to the IP address of your server.
* Ansible >= 2.9.6 with installed roles: \
  * geerlingguy.firewall
  * geerlingguy.certbot
  * udelarinterior.jitsi_meet

Install this roles with `ansible-galaxy role install -r requirements.yml`.

## Provisioning

Boot to the rescue system (with your public key selected) and remember the password.

### Setup Adaptec hardware raid controller [optional]

Create RAID 1 array:

1. `arcconf DELETE 1 LOGICALDRIVE ALL`
1. `arcconf TASK START 1 DEVICE ALL INITIALIZE`
1. `arcconf CREATE 1 LOGICALDRIVE MAX 1 0 0 0 1 noprompt`

see result: `arcconf GETCONFIG 1 LD`
more information: https://wiki.hetzner.de/index.php/Adaptec_RAID_Controller

### Execute Ansible

Basic server setup:
1. Install server image: `ansible-playbook --extra-vars "target=hqrs1" install_image.yml`
1. `ssh-copy-id -i ~/.ssh/id_rsa.pub root@hqrs1` (see Issues #1). Test with `ssh root@hqrs1`. Perhaps you want to change the root password now.
1. Ansible needs Phyton: `ansible-playbook --extra-vars "target=hqrs1" install_python.yml`
1. Update server packages, create user(s) and setup sshd: `ansible-playbook --extra-vars "target=hqrs1" bootstrap.yml`

Final setup: `ansible-playbook -K hqrs1.yml` (this requires the admin user password).

*Notice:\
It should be possible to provision every root server, just skip the first step.

## Issues:

1. Hetzners `installimage ... -t yes` doens't copy the ssh key files from the rescue system to the new installed system. So the execution of the command `ssh-copy-id` is needed.
1. When you reconfigure your firewall (e.g. adding/removing ports) try running `service firewall restart` when it comes to errors or `iptables -L` doesn't what you expected. As I always say: it's always up to the cache or the firewall when it comes to errors.

## Appendix

The Hetzner `installimage` help:

```
usage:  installimage [options]

  without any options, installimage starts in interactive mode.
  possible options are:

  -h                    display this help

  -a                    automatic mode / batch mode - use this in combination
                        with the options below to install without further
                        interaction. there will be no further confirmations
                        for deleting disks / all your data, so use with care!

  -c <configfile>       use the specified config file in
                        /root/.oldroot/nfs/install/configs for autosetup. when using
                        this option, no other options except '-a' are accepted.

  -x <post-install>     Use this file as post-install script, that will be executed after
                        installation inside the chroot.

  -n <hostname>         set the specified hostNAME.
  -r <yes|no>           activate software RAID or not.
  -l <0|1|5|6|10>       set the specified raid LEVEL.
  -i <imagepath>        use the specified IMAGE to install (full path to the OS image)
                        - supported image sources: local dir, ftp, http, nfs
                        - supported image types: tar,tar.gz,tar.bz,tar.bz2,tar.xz,tgz,tbz,txz
                        - supported binary image types: bin,bin.bz2 (CoreOS only)
                        examples:
                        - local: /path/to/image/filename.tar.gz
                        - ftp:   ftp://<user>:<password>@hostname/path/to/image/filename.tar.bz2
                        - http:  http://<user>:<password>@hostname/path/to/image/filename.tbz
                        - https: https://<user>:<password>@hostname/path/to/image/filename.tbz
                        - nfs:   hostname:/path/to/image/filename.tgz
  -g                    Use this to force validation of the image file with detached GPG signature.
                        If the image is not valid, the installation will abort.
  -p <partitions>       define the PARTITIONS to create, example:
                        - regular partitions:  swap:swap:4G,/:ext3:all
                        - lvm setup example:   /boot:ext2:256M,lvm:vg0:all
  -v <logical volumes>  define the logical VOLUMES you want to be created
                        - example: vg0:root:/:ext3:20G,vg0:swap:swap:swap:4G
  -d <drives>           list of hardDRIVES to use, e.g.:  sda  or  sda,sdb
  -f <yes|no>           FORMAT the second drive (if not used for raid)?
  -s <de|en>            Language to use for different things (e.g.PLESK)
  -z PLESK_<Version>    Install optional software like PLESK with version <Version>
  -K <path/url>         Install SSH-Keys from file/URL
  -t <yes|no>           Take over rescue system SSH public keys
  -u <yes|no>           Allow usb drives
  -G <yes|no>           Generate new SSH host keys (default: yes)
```