# k3s w/ LINSTOR Ansible Playbook

Build a lightweight k3s cluster w/ LINSTOR installed using Ansible.

Preqs:

  - An customer or active evaluation account for https://my.linbit.com (contact sales@linbit.com).
  - Deployment environment must have Ansible `2.7.0+` and `python-netaddr`.
  - Playbook currently only supports Ubuntu Focal hosts.
  - One or more hosts each with an addition unused block device attached.

# Usage

Adjust the variables in `hosts.ini` accordingly. For example:
```
... snip ...
# all vars
[multi:vars]
ansible_user=root
become=yes
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
# Linstor variables
drbd_backing_disk=/dev/sdb
drbd_replication_network=192.168.222.0/24
# my.linbit.com customer portal credentials
lb_user="fthomas"
lb_pass="Password1!"
```
You can only list a single host as a `k3s_master` and less or equal to 31 hosts as a `k3s_worker`.

Bring up cluster using Vagrant and run Ansible:

```sh
ansible-playbook -i hosts.ini playbook.yaml
```
You can bring up a K3s cluster without LINSTOR via `--skip-tags linstor`. Or, you can install LINSTOR into an existing kubernetes cluster using `--tags linstor`.

If you don't want to put LINBIT credentials into your `host.ini`, you can pass extra args on the command line instead:

```sh
ansible-playbook -e lb_user="username" -e lb_pass="password" playbook.yaml
```

# Testing Installation

Shell into the `k3s_master` and watch pods coming up:

```sh
watch kubectl get pods -n linstor
```

Once deployed, there are preconfigured storage classes following the named, `linstor-csi-<storagepool>-r<replicas>`, where `<storagepool>` is the name of the LINSTOR storage pool, and `<replicas>` is the number of physical replicas LINSTOR will replciate to.

```sh
kubectl get sc
```

# To Do

  - Support more distros.
  - Use `drbd_replication_network` var to define LINSTOR replication network.
