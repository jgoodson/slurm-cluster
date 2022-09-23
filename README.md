### Steps

- Create a Proxmox hypervisor with some utility VMs
    - Create a virtual router for your VM subnet, I used VyOS
    - Configure DNS forwarding to IPA master (10.0.1.1) for `cluster.local`
    - Create a utility VM on the VM subnet with ansible and proxmoxer
- Configure a template
    - I used the Rocky 8 qcow2 image with cloud-init
    - I did some minor tweaks to the `/etc/yum.conf` and cloud-init settings
    - I created an LXC container with apt-cacher-ng to keep the repeated yum updates local
- Install Ansible roles/collections
    - `ansible-galaxy install scicore.slurm`
    - `ansible-galaxy collection install freeipa.ansible_freeipa`
    - `ansible-galaxy install geerlingguy.nfs`
- Provision VMs
    - `ansible-playbook -v provision_vm.yml`
        - Autogens inventory directory for later playbooks
- Install FreeIPA
    - `ansible-playbook -v -i vm_inventory/ install_freeipa.yml`
        - Installs FreeIPA server on the server/replica nodes and configures clients
- Configure Slurm with scicore playbook
    - `ansible-playbook -v -i vm_inventory/ configure_slurm.yml`
        - Sets up a DNS SRV record and builds a slurm cluster with "configless" clients
- Set up NFS network automounts
    - `ansible-playbook -v -i vm_inventory/ config_nfs.yml`
        - Installs and configures NFS with Kerberos authentication on both server and clients
        - Utilizes FreeIPA SSSD distribution of automounts for /mnt/data and /home

### Todo

- Determine IP address from machine to allow DHCP
    - Alternatively, configure a DNS server outside of FreeIPA to resolve DHCP VMs, somehow?
- Deal with pam_mkhomedir integration with automounted homedir
    - Alternatively, somehow make IPA user creation make an NFS homedir for new users so it doesn't have to be done manually
- CLOOOOUUUUUUUD
    - Setup a cluster with ParallelCluster, Bath, or some other tool and integrate it with this one