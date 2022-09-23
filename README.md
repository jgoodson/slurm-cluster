### Steps

- Provision VMs
    - Setup Proxmox
    - Install and configure VyOS router VM on virtual network
    - Configure DNS forwarding to IPA master (10.0.1.1) for `cluster.local`
    - `ansible-playbook -v provision_vm.yml`
    - Autogens inventory directory for later playbooks
- Install FreeIPA
    - `ansible-playbook -v -i vm_inventory/ install_freeipa.yml`
- Configure Slurm SRV record for "configless" slurm workers
    - Add `_slurmctld._tcp` to IPA DNS zone `0 0 6817 slurmd.cluster.local.`
- Configure Slurm with scicore playbook
    - `ansible-playbook -v -i vm_inventory/ configure_slurm.yml`