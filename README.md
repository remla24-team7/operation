# operation

Before you get started make sure to `dvc pull` the [model.dvc](model.dvc).

## A2: Docker Compose

We provide three Compose configurations:

`docker compose up` to use our image releases.

`docker compose -f docker-compose.git.yml up --build` to build from Dockerfiles on the remote `main` branches.

`docker compose -f docker-compose.local.yml up --build` to build from Dockerfiles in locally cloned repositories.

## A3: Vagrant/Kubernetes

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

Make sure the model folder (`model/model.keras`, `model/tokenizer.joblib`, `model/encoder.joblib`) is in `${VAGRANT_SYNCED_FOLDER:-.}`.

`kubectl apply -f kubernetes/app.yml`

`kubectl apply -f kubernetes/model-service.yml`

> Note: [unless AVX instructions are available inside the VMs, tensorflow and therefore model-service will crash](https://stackoverflow.com/questions/65780506/how-to-enable-avx-avx2-in-virtualbox-6-1-16-with-ubuntu-20-04-64bit).

### Start monitoring the application

    kubectl apply -f kubernetes/monitoring/monitoring_example.yml

This should deploy three pods distributed on the controller, node1 and node2. Open the Prometheus dashboard and move on to Status >> Targets to find the metrics endpoint. 

To add the dashboard, navigate to the Grafana dashboard at http://192.168.56.110:30020. 

To obtain the password for the Grafana dashboard run the following command:

    kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

The login credentials are:

login: admin

password: *generated from command above*

Now add Prometheus as a data source (Configuration > Data sources), the URL for Prometheus is http://192.168.56.110:30010

And add the JSON dashboard located in `kubernetes/monitoring/remla24-team7-dashboard.json`

## A5: Istio Service Mesh
### Running instructions

```
minikube start
istioctl install
kubectl apply -f {istio install}/samples/addons/prometheus.yaml
kubectl apply -f {istio install}/samples/addons/jaeger.yaml
kubectl apply -f {istio install}/samples/addons/kiali.yaml
kubectl apply -f kubernetes/launch.yml

minikube tunnel
```

To then open the kiali dashboard

```
istioctl dashboard kiali
```
