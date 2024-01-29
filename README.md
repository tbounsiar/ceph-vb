# From your host machine

Clone vagrant project
```shell
git clone https://github.com/tbounsiar/ceph-vb.git
```

Install vagrant hostmanager plugin
```shell
cd ceph-hv
vagrant plugin install hostmanager
```

```shell
vagrant up
```

# From machine where you run ansible
https://docs.ceph.com/projects/ceph-ansible/en/latest/

````shell
git clone https://github.com/ceph/ceph-ansible.git
cd ceph-ansible
git checkout stable-7.0
pip install -r requirements.txt

sudo add-apt-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible

ansible-galaxy install -r requirements.yml
cp site.yml.sample site.yml

cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
````

## Uncomment and Make updates on file 
Update `all.yml`
```yml
ceph_origin: repository
ceph_repository: community
ceph_stable_release: pacific
monitor_interface: eth0
journal_size: 1024
public_network: 192.168.11.0/24
cluster_network: 192.168.10.0/24
dashboard_enabled: True
dashboard_admin_user: admin
dashboard_admin_password: p@ssw0rd
grafana_admin_user: admin
grafana_admin_password: admin
```

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
copy inventories dir from ceph-hv to ceph-ansible

````shell
ansible -i inventories/hyperv all -m ping
ansible-playbook -i inventories/hyperv site.yml -v
````