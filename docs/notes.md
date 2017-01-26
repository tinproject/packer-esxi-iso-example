# Image creation with packer

https://www.packer.io/docs


## allow trafic on VMWare ESXI server for vnc

http://t3chnot3s.blogspot.com.es/2012/03/how-to-enable-vnc-access-to-vms-on.html

Add the rules to `service.xml`, with the number one more of the current rules.
Refresh the firewall ruleset

http://www.metallic-badger.com/using-packer-deploy-vms-nested-esxi-environment/

This changes do not persist on restart. Some people suggest enable `gdbdebugger` rule to allow traffic, but I don like as it is a very permissive firewall rule. 

http://www.bluebox-web.com/2013/04/11/create-a-custom-esxi-firewall-service/

The solution to create a permanent firewall rule is to include the rule in a _vib_ package. The tool to create custom _vib_ packages: 'Fling' is now deprecated.  

** The best solution I've come to this is to enable the vnc rule prior run packer. I create the `enable_vnc_on_esxi_firewall.yml` and `disable_vnc_on_esxi_firewall.yml` to enable vnc on ESXi host. This changes don't persist at restart.

## Allow key-based ssh access on esxi host

From https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1002866

> On the remote host, store the public key content, id_rsa.pub in ~/.ssh/authorized_keys.
>
> Notes:
> 
>     For ESXi 5.x, 6.0 and 6.5 the location of authorized_keys is: /etc/ssh/keys-<username>/authorized_keys
>     More than one key can be stored in this file.
> 
> To allow root access, change PermitRootLogin no to PermitRootLogin yes in the /etc/ssh/sshd_config file.
> To disable password login, ensure that ChallengeResponseAuthentication and PasswordAuthentication are set to no.

For now only non-passphrase keys works. See: https://github.com/mitchellh/packer/issues/3602

Only works for ´root´ user.

## ESXi disk types:

- zeroedthick
- thin
- eagerzeroedthick
- rdmp
- rdm
- 2gbsparse / sparse2GB

See: https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1028042

## ESXi guest host identifiers:

- debian8-64
- ubuntu-64

** In case of doubt, the best is to create a vm manually and then extract the values from the .vmx file.

## Other VM settings

I choose to use paravirtualized drivers: vmxnet3, pvscsi


## Command line commands to register the VM

https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2012964
http://www.blackmanticore.com/206da0c17840b37177b113a44d96e1c4

** not needed telling packer to not unregister the vm. 

## Boot command

Examples of different packer templates with different linux flavours: https://github.com/chef/bento

### boot command for Debian

```yml
    boot_command:
      - "PACKER_KEY_INTERVAL=80ms"
      - "<esc><wait>"
      - "install <wait>"
      - "preseed/url=http://{% raw %}{{.HTTPIP }}:{{ .HTTPPort }}{% endraw %}/{{ preseed_file }} <wait>"
      - "debian-installer=en_US <wait>"
      - "auto <wait>"
      - "locale=en_US <wait>"
      - "kbd-chooser/method=es <wait>"
      - "keyboard-configuration/xkb-keymap=es <wait>"
      - "netcfg/get_hostname={{ hostname }} <wait>"
      - "netcfg/get_domain={{ domain }} <wait>"
      - "fb=false <wait>"
      - "debconf/frontend=noninteractive <wait>"
      - "console-setup/ask_detect=false <wait>"
      - "console-keymaps-at/keymap=es <wait>"
      - "grub-installer/bootdev=/dev/sda <wait>"
      - "<enter><wait>"
```

### boot command for Ubuntu

```yml
    boot_command:
      - "PACKER_KEY_INTERVAL=80ms"
      - "<enter><wait><f6><esc><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>"
      - "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>"
      - "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>"
      - "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>"
      - "install <wait>"
      - "auto "
      - "url=http://{% raw %}{{.HTTPIP }}:{{ .HTTPPort }}{% endraw %}/{{ preseed_file }} "
      - "preseed-md5={{ generated_preseed.stat.md5 }} "
      - "locale=en_US.UTF-8 "
      - "language=en "
      - "country=US "
      - "keymap=es "
      - "hostname={{ hostname }} "
      - "domain={{ domain }} "
      - "fb=false "
      - "initrd=/install/initrd.gz "
      - "debconf/frontend=noninteractive "
      - "console-setup/ask_detect=false "
      - "console-keymaps-at/keymap=es "
      - "grub-installer/bootdev=/dev/sda "
      - "<enter><wait>"
```

## Preseed files for Ubuntu / Debian

Preseed files are the way to unattend the instalation of Debian/Ubuntu, consists of a series of answer to the installation process asks.

The process is not very well documented. Mostly examples and not proper documentation.

Some resources:
- https://www.debian.org/releases/jessie/amd64/index.html.en
- https://www.debian.org/releases/jessie/amd64/apb.html.en
- https://help.ubuntu.com/lts/installation-guide/amd64/index.html
- https://help.ubuntu.com/lts/installation-guide/amd64/apb.html
- https://wikitech.wikimedia.org/wiki/PartMan
- http://askubuntu.com/questions/806820/how-do-i-create-a-completely-unattended-install-of-ubuntu-desktop-16-04-1-lts

### Sources of information

To get the proper values for some options of preseed files can install manually the system and then copy files generated by the install process.

Run an installation, on the installed machine run:
```bash
sudo apt-get install debconf-utils
sudo debconf-get-selections --installer > system.preseed
```

The install process logs to the folder: ´/var/log/installer´

Also more information lie on ´/var/cache/debconf´

### Gotchas: Can't choose whatever language/locale

In my case I try to use language _en_ with country _ES_ and I have to desist as always ended prompting a dialog.

You cant choose a combination of language/country that has non existing locale.

Lots of bugs rising that matter:
https://bugs.launchpad.net/ubuntu/+source/localechooser/+bug/37138
https://bugs.launchpad.net/ubuntu/+source/localechooser/+bug/828412

http://www.gnu.org/software/gettext/manual/gettext.html#Setting-the-POSIX-Locale

### Gotchas: Preseed partitioning, not for multiple disk

> Debian preseed's partman options are an incomprehensible automatic partitioning language. 

** Preseed don't allow to partition multiple disks unless use lvm.


