This repo is created for the creation and unattended installation of linux virtual 'pets' on an EXSi server in my home lab.

It uses Ansible, Packer, and the VMWare ESXi server.

The goal is to have a way of create a virtual machine and install a linux distribution in an unattended way from ISO files. The intend is to create a minimal running version and then provisioning the machine with other tools.

### How it works

Ansible is used to create the templates needed to run packer. 

In this case of the example I want a new pet named _howler_, the playbook that creates the packer templates is `create_server_howler.yml`. Every server has to have it's own playbook for the task (simply a copy with the correct data).

The data needed to generate the packer templates is on the `vm_data` folder. The file `howler_packer_template.yml` contains a yaml version of the JSON packer template, I feel that is easier to write and better to automate.
  
Also there is the _preseed_ template `howler_preseed_template.j2` that creates the preseed file needed for Debian/Ubuntu installatión.

At the end of the playbook is a task to run packer, but I prefer to run it manually. It's a long process and I prefer to see packer output.
 
Also there is two other playbooks `enable_vnc_on_esxi_firewall.yml` and `disable_vnc_on_esxi_firewall.yml` that enable and disable VNC ports on the ESXi firewall. Note that packer needs to access the ESXi server vía ssh.


### Gotchas / Caveats / Lessons learned

- I take this approach because I only have the free licence of the ESXi hypervisor and not VSphere Center, so no easy possibility to deal with vm templates or automation.
- Also this is for _pets_, and my needs of new virtual machines is scarce. 
- To connect to an ESXi server with the packer `vmware-iso` builder you need ssh access, it could be a security concern, most knowing that ssh on the ESXi server only works with the root account and packer don't support password protected keys.
- The management of firewall rules on the ESXi server is tricky, I believe surely is better using vSphere Center but this is what I have. For a new virtualization server I probably go with OpenStack.
- The `vmx` file options are not very well documented, and you'll need it because the packer 'sane defaults' vmx template is not sane for use with ESXi. The best is to create a vm manually with the same characteristics you want and then copy the `.vmx` file from the datastore and use this options.
- The preseed process is also not very well documented, documentation are a bunch of examples not a proper reference documentation.
- The preseed files consists in a series of answers to what the debian-installer ask at installation time.
- You can't get to work a language/country combination that didn't have a proper locale. For example, I cannot say that my server is on Spain (ES) and I want to use it on English (en). It's suppose to work but is not, and they didn't tell you so you end up loosing a lot of time if you try.
- Also preseed can't partition multiple disks (unless you use lvm). I plan to use multiple disks butI have to go with one disk, hope I choose tha size of partitions well.

- The time involved in investigate the preseed files and the whole process of automated installation is not worth it for a few machines. In the same time span I could surely install ~20 machine manually, more than I'll ever need.
- On the contrary it worths the time to learn how thing works and to have a documented and ready way to restore the machines in case of disk or other failure.
- Use Packer in a cloud environment is way more easy: take an image, modify it and write to a new image, not need to deal with installation processes.

- more info on [docs/notes.md](docs/notes.md)


In the end it was a good time learning. I'll share this hoping this example serve to other people and complements the current documentation. 
