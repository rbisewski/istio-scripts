# istio-scripts

A number of sample scripts for using Istio, as well as a guide to installing
Istio with the latest kubectl and helm 3.

### Environment setup

This guide assumes that the kubeconfig is stored in the following location:

```
/kubernetes/kube_config_cluster.yml
```

In addition, some simple aliases are added to make these commands more compact:

```bash
alias k8s='kubectl --kubeconfig=/kubernetes/kube_config_cluster.yml'
alias helm-w-k8s='helm --kubeconfig=/kubernetes/kube_config_cluster.yml'
```

### Installing Istio

First clone the istio into a folder, then link the binary to `/usr/local/bin/` like so:

```bash
cd /kubernetes/
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.3.0 sh -
sudo ln -s /kubernetes/istio-1.3.0/bin/istioctl /usr/local/bin/istioctl
```

Then install the latest version of helm 3 (without tiller).

```bash
wget https://get.helm.sh/helm-v3.0.0-beta.3-linux-amd64.tar.gz
tar -zxvf helm-v3.0.0-beta.3-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/
```

Add the Istio.io repo to helm:

```bash
helm-w-k8s repo update
helm-w-k8s repo add istio.io https://storage.googleapis.com/istio-release/releases/1.3.0/charts/
```

Create a dedicated namespace for istio:

```bash
k8s create namespace istio-system
```

Go to the Istio.io directory and then install the helm script:

```bash
cd /kubernetes/istio-1.3.0/
helm-w-k8s template install/kubernetes/helm/istio-init --name-template istio-init --namespace istio-system | k8s apply -f -
```

Verify that all 23 Istio CRDs were committed to the Kubernetes api-server using the following command:

```bash
k8s get crds | grep 'istio.io' | wc -l
```

Then go ahead and install the default install profile:

```bash
cd /kubernetes/istio-1.3.0
helm-w-k8s install install/kubernetes/helm/istio --name-template istio --namespace istio-system
```

If successful, you will see the following message:

```
Your release is named Istio.

To get started running application with Istio, execute the following steps:
1. Label namespace that application object will be deployed to by the following command (take default namespace as an example)

$ kubectl label namespace default istio-injection=enabled
$ kubectl get namespace -L istio-injection

2. Deploy your applications

$ kubectl apply -f <your-application>.yaml

For more information on running Istio, visit:
https://istio.io/
```

Consider testing to see that Istio services and dedicated pods are running:

```
k8s get svc -n istio-system
k8s get pods -n istio-system
```
