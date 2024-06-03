# operation

## A3: Kubernetes

### Provision the VMs

`vagrant up`

Vagrant creates the VMs according to the [Vagrantfile](Vagrantfile) and uses the [Ansible playbook](ansible/playbook.yml) to set up a Kubernetes (k3s) cluster with Helm, Prometheus, Grafana, and Kubernetes Dashboard installed.

The `kubeconfig` should be available on the host at `${VAGRANT_SYNCED_FOLDER:-.}/k3s.yaml`. Either set the `KUBECONFIG` environment variable or use `--kubeconfig` flags in subsequent commands.

### Expose the dashboards

`kubectl apply -f kubernetes/monitoring/expose-dashboards.yml`

- Prometheus should be reachable at http://192.168.56.110:30010
- Grafana should be reachable at http://192.168.56.110:30020
- Kubernetes Dashboard should be reachable at https://192.168.56.110:30030

### Create the ClusterAdmin user and generate a Bearer token to access Kubernetes Dashboard

`kubectl apply -f kubernetes/monitoring/cluster-admin.yml`

`kubectl -n monitoring create token admin`

### Deploy the `app` and `model-service`

Make an entry in `/etc/hosts`: `192.168.56.110 app.remla.local`

Make sure the model files (`model.h5`, `tokenizer.joblib`, `encoder.joblib`) are in `${VAGRANT_SYNCED_FOLDER:-.}`.

`kubectl apply -f kubernetes/app.yml`

`kubectl apply -f kubernetes/model-service.yml`

### ...watch `model-service` keep crashing

```
➜ kubectl run test --image=ghcr.io/remla24-team7/model-service:0.1.0 --stdin --tty /bin/bash
If you don't see a command prompt, try pressing enter.
root@test:/app# ls
model_service  poetry.lock  pyproject.toml
root@test:/app# poetry run python
Python 3.11.0rc1 (main, Aug 12 2022, 10:02:14) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow
Illegal instruction (core dumped)
```

https://stackoverflow.com/questions/49094597/illegal-instruction-core-dumped-after-running-import-tensorflow

### Start monitoring the application

`kubectl apply -f kubernetes/monitoring/monitoring_example.yml`

This should deploy three pods distributed on the controller, node1 and node2. Open the Prometheus dashboard and move on to Status >> Targets to find the metrics endpoint. 

To add the dashboard, navigate to the Grafana dashboard at http://192.168.56.110:30020. 

login: admin

password: prom-operator

And add the JSON dashboard located in `kubernetes/monitoring/remla24-team7-dashboard.json`