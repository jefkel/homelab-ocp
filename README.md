# jk-ocp-lab

Home OCP Cluster build

## Deployment Model

The deployment is a 4-node cluster hosted on a single ESXi host consisting of 1 master, 1 infra and 2 compute nodes.

### VM Deploy and Configure

Specs:
 - 4 RedHat VM's:
   - 1 x 2G RAM, 2 vCPU, 20GB Thin Provisioned storage
   - 3 x 2G RAM, 1 vCPU, 20GB Thin Provisioned storage

Split DNS (internal as well as external DNS)
Static Leases for hosts via existing DHCP service
check on DHCP DNS host (had issue with pointing to external DNS which messes up internal network DNS)
check external access to API allowed in firewall (or re-configure inventory for internal only)

Each VM built with Minimal CentOS image
 - root and user (admin) accounts provisioned during install
 - mm01, mnode01, mnode02, mnode03 in the cavenet.ca domain for hostnames

Run homelab/playbooks/deployOS.yml playbook to do initial server setup and other prep steps
 - notes: 
   - no sudo without password
   - no key'd login (yet)
eg: (to bypass sudo and key issues until after node prep)
ansible-playbook playbooks/deployOS.yml -K -k


### OpenShift Deployment

Use the Advanced Installation (Ansible) scripts and configuration steps with some key additions to the ansible host configuration:
run:
ansible-playbook ../openshift-ansible/playbooks/prerequisites.yml
then
ansible-playbook ../openshift-ansible/playbooks/deploy_cluster.yml

- openshift_master_default_subdomain=app.cavenet.ca
- MasterPublicURL=https://mm01.cavenet.ca:8443
- os_firewall_use_firewalld=true

Very important to set the following to override the default minimum requirements!
- openshift_check_min_host_memory_gb=1.5
- openshift_check_min_host_disk_gb=15

See the actual hosts.OS file for a full picture of the additional options, but the above were the main requirements.

### TroubleShooting Fun
Check your DHCP settings for your home network (external DNS will break your install and be more annoying to troubleshoot than a cracked wire in a car stereo wiring harness.)

Firewalls, port-forwarding, provider port blocking, and other networking fun when hosting presentation material from home.

Todo:
VM Snapshots are awesome, but full deploy automation is better!

## Application Deployments

