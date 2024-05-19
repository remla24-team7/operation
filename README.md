# operation

## A3: Kubernetes

### Provision the VMs

`vagrant up`

Vagrant creates the VMs according to the [Vagrantfile](Vagrantfile) and uses the [Ansible playbook](ansible/playbook.yml) to set up a Kubernetes (k3s) cluster with Helm, Prometheus, Grafana, and Kubernetes Dashboard installed.

### Expose the dashboards

`kubectl --kubeconfig ${VAGRANT_SHARED_FOLDER:-.}/k3s.yaml apply -f kubernetes/monitoring/expose-dashboards.yml`

You should be able to navigate to `192.168.56.110:300{10,20,30}` (ports are configured in [the manifest](kubernetes/monitoring/expose-dashboards.yml)).