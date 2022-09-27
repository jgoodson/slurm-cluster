### Steps

- Create a Proxmox hypervisor with some utility VMs
    - Add an "ansible" user for API use
    - Create a virtual router for your VM subnet, I used VyOS
    - Configure DNS forwarding to IPA master (10.0.1.1) for `cluster.local`
    - Create a utility VM on the VM subnet with ansible and proxmoxer
- Configure a template
    - I used the Rocky 8 qcow2 image with cloud-init
    - I did some minor tweaks to the `/etc/yum.conf` and cloud-init settings
    - I created an LXC container with apt-cacher-ng to keep the repeated yum updates local
- Install Ansible roles/collections
    - `ansible-galaxy install -r requirements.yml`
- Put the necessary secrets in `vm_inventory/group_vars/all`
    - `mkdir -pv vm_inventory/group_vars`
    - `ansible-vault create vm_inventory/group_vars/all`
        - `api_password` - Proxmox password for "ansible@pam" by default
        - `ipaadmin_password` - FreeIPA Admin password
        - `ipadm_password` - FreeIPA directory manager password
        - `slurm_slurmdbd_mysql_password` - Slurm MySQL DB Password
- Provision VMs
    - `ansible-playbook -v provision_vms.yml`
        - Autogens inventory directory for later playbooks
- Install FreeIPA identity management
    - `ansible-playbook -v install_freeipa.yml`
        - Installs FreeIPA server on the server/replica nodes and configures clients
- Configure Slurm with scicore playbook
    - `ansible-playbook -v configure_slurm.yml`
        - Sets up a DNS SRV record and builds a slurm cluster with "configless" clients
- Set up NFS network automounts
    - Add a VirtualIO drive to the nfs1 VM for storage
        - Cannot be automated with existing proxmox roles. Would need to add my own.
    - `ansible-playbook -v configure_nfs.yml`
        - Sets up storage and installs NFS server
        - Configures NFS with Kerberos authentication on both server and clients
        - Sets up FreeIPA SSSD distribution of automounts for /mnt/data and /home


### Todo

- CLOOOOUUUUUUUD
    - Setup a cluster with ParallelCluster, Bath, or some other tool and integrate it with this one
- Automate more things - low priority
    - Promox setup?
    - Router/utility box provisioning?
    - Figure out how to automate creation of a second VHD (proxmox_kvm will not do it)
- Clean up template and make cloud-init work properly
    - Right now all my hosts have the same SSH host keys because my change to cloud-init config didn't do what I thought it would
    - I need to update the template but it is a bit of a pain
- Figure out why sometimes the DNS records don't get created for hosts (not consistent which ones)
    - Incomplete client addition process?
    - Running the `install_freeipa.yml` playbook  again seems to fix this.
