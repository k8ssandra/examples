# K8ssandra Operator Demo

This demo installs the following stack on Kubernetes:

* Cassandra 4.0
* Stargate
* Reaper
* Medusa
* Prometheus
* Grafana

This demo utilizes Kustomize (via `kubectl -k`) to install all components. With that in mind you can easily use this repository as a base and provide your own customizations.

## 1. Validate Connectivity

Before getting started validate you have `kubectl` installed and it is connected to a running Kubernetes cluster.

```console
kubectl version
Client Version: v1.24.2
Kustomize Version: v4.5.4
Server Version: v1.22.8-gke.202

kubectl cluster-info
Kubernetes control plane is running at https://34.66.92.229
GLBCDefaultBackend is running at https://34.66.92.229/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
KubeDNS is running at https://34.66.92.229/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://34.66.92.229/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

## 2. Install K8ssandra

All components are currently specified in the `base/` directory which may be applied to your cluster with 

`kubectl apply --server-side --force-conflicts -k base`

alternatively you may leverage one of the existing overlays (in the `overlays/` directory) which have customizations to match the particular environment. For example when deploying on GKE use

`kubectl apply --server-side --force-conflicts -k overlays/gcp`

this will utilize GKE specific components like the `standard-rwo` storage class.

## 3. Validate Deployment

When everything is finished installing you should see a number of pods running in the `default` namespace representing the Stargate and C* nodes.

```console
kubectl get pods
NAME                                                    READY   STATUS    RESTARTS   AGE
demo-dc1-default-stargate-deployment-84c9d49448-c5k8g   1/1     Running   0          6h13m
demo-dc1-default-sts-0                                  2/2     Running   0          6h17m
demo-dc1-default-sts-1                                  2/2     Running   0          6h17m
demo-dc1-default-sts-2                                  2/2     Running   0          6h17m
```

## 4. Access Grafana

To access the web interface for Grafana you must port-forward to the cluster.

```console
kubectl port-forward -n monitoring svc/grafana 3000:3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

From here open your browser to [http://127.0.0.1:3000/http://127.0.0.1:3000/). The username and password are `admin`. You will find Cassandra specific dashboards have been preinstalled and nodes are automatically discovered.

## _Optional:_ Deploy without Checkout

Kustomize provides support for deployment without checking out this repo. Simply run

```console
kubectl apply --server-side --force-conflicts -k "https://github.com/k8ssandra/examples//k8ssandra-operator/overlays/gcp
?ref=main"
```

and it will pull down the appropriate manifests and apply them to the cluster.
