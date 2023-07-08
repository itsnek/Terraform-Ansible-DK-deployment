# Terraform-Ansible deployment of Docker-Kubernetes in a Virtualbox vm

## Terraform 

### Content 
- keys/vagrant, ssh keys to connect to the vm.
- provider.tf
- main.tf
- variables.tf, where you can set global variables that may be used in main.tf.

### Steps
In terraform folder run:
- `terraform init -upgrade`
- `terraform plan`
- `terraform apply` (you can use --auto-approve for skipping aproval part)


## Docker-Kubernetes deployment using ansible

### System specs for each node
- At least 2 core processors
- At least 2 gb RAM mem
- At least 50gb disk mem space(especially in master node)

### Components
- 1 master node
- 1 worker node

### Steps
1. Configure hosts.yml to contain the appropriate ip addresses, as well as users for the nodes. In this case we have created an explicit docker/kubernetes user, "duser/kuser", with sudo priviledges, by executing the command: ` ansible-playbook create-users.yml -i hosts.yml -K `
2. (Optional) Modify the dkb-playbook.yml to add any extra needed system package.
3. Make sure that you can perform a ssh connection to the nodes.
4. Either do "eval \`ssh-agent\` && ssh-ad" or add --ask-pass in the next command(it may need ssh-pass package).
5. To execute the docker-kubernetes deployment, run the command:
   ` ansible-playbook dkb-playbook.yml -i hosts.yml -K `
6. The above command except from installing the appropriate versions of kubelet, kubeadm and kubectl also disables swap off, deploy default containerd configuration and restarts. If you want to turn swap off permanantly, modify and comment lines in /etc/fstab. The playbook initialize cluster by adding cni provider and also restarts kubelet service to apply cni configuration.
7. After applying cni-plugin, check whether you need to untaint taints from the node. Cluster should now be ready.
8. To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set:
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    ...
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
        
9. Last task is to join worker node in the cluster(use the same user-passwd, cause join-command is sustained only in one session).