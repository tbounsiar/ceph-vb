# Installing a Ceph cluster using Vagrant and VirtualBox

## Introduction
Ceph is an open-source, distributed storage system designed for scalability, reliability, and performance. 
It provides object storage, block storage, and file storage services in a single platform. 

Video: https://www.youtube.com/watch?v=-yQKiqKUw70&t=23s

## Prerequisites
Before getting started, ensure that you have the following prerequisites installed on your machine:

**VirtualBox:** A free and open-source hosted hypervisor. You can download it from [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads).

**Vagrant:** An open-source tool for building and managing virtualized development environments. Download it from [Vagrant Downloads](https://developer.hashicorp.com/vagrant/downloads).

## From your host machine
Clone vagrant project
```shell
git clone https://github.com/tbounsiar/ceph-vb.git
```
Install vagrant hostmanager plugin
```shell
cd ceph-vb
vagrant plugin install hostmanager
vagrant up
```

## From machine where you run ansible

Link to the Ceph Ansible documentation
https://docs.ceph.com/projects/ceph-ansible/en/latest/

### Prepare the environment
```shell
# Clone ceph-ansible and choose you version
git clone https://github.com/ceph/ceph-ansible.git
cd ceph-ansible
git checkout stable-7.0

# Install pip
sudo apt update
sudo apt install python3-pip

# Install pip requirements
pip install -r requirements.txt

# Install ansible
sudo add-apt-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible

# Install ansible-galaxy requirements
ansible-galaxy install -r requirements.yml

# Copy the needed files
cp site.yml.sample site.yml
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
```

### Uncomment and make updates on files

#### Update `all.yml`
```yml
ceph_origin: repository
ceph_repository: community
ceph_stable_release: pacific
monitor_interface: eth1
journal_size: 1024
public_network: 192.168.11.0/24
cluster_network: 192.168.10.0/24
dashboard_enabled: True
dashboard_admin_user: admin
dashboard_admin_password: p@ssw0rd
grafana_admin_user: admin
grafana_admin_password: admin
```

#### Update file `osds.yml`
run this command on each nodes to verify disks paths in all osds
```shell
lsblk
```
Update file `osds.yml`
```yml
devices:
  - /dev/sdb
  - /dev/sdc
```

Copy inventories dir from ceph-vb to ceph-ansible

```shell
ansible -i inventories/vbox all -m ping
ansible-playbook -i inventories/vbox site.yml -v
```
### Verify Ceph Cluster

After the Ansible playbooks have completed, verify the status of your Ceph cluster.

You can use Ceph Dashboard https://node-4:8443

For Grafana https://node-5:3000