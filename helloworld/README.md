# Istio Hello World Example

A very basic Istio script that deploys a service which does nothing but setup
two instances of a Hello World image.

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

### Installation

Install the scripts as you would a normal Kubernetes deployment:

```bash
k8s apply -f helloworld.yaml
k8s apply -f helloworld-gateway.yaml
```

Describe the service to figure out which IPs it has deployed to:

```bash
k8s describe svc helloworld
```

Which ought to yield something like the following:

```bash
Name:              helloworld
Namespace:         default
Labels:            app=helloworld
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"helloworld"},"name":"helloworld","namespace":"default"},...
Selector:          app=helloworld
Type:              ClusterIP
IP:                10.43.208.7
Port:              http  5000/TCP
TargetPort:        5000/TCP
Endpoints:         10.42.0.152:5000,10.42.0.153:5000
Session Affinity:  None
Events:            <none>
```

The IPs associated with the service are:

* 10.42.0.152:5000
* 10.42.0.153:5000

So attempt to curl them to see if they are working:

```bash
curl 10.42.0.152:5000/hello
curl 10.42.0.153:5000/hello
```

### Autoscaling

To enable autoscaling across both instances, run the following commands:

```bash
k8s autoscale deployment helloworld-v1 --cpu-percent=50 --min=1 --max=10
k8s autoscale deployment helloworld-v2 --cpu-percent=50 --min=1 --max=10
```

Then check the Horizontal Pod Autoscaler (HPA) to check the scaler summary:

```bash
k8s get hpa
```

### Testing the autoscaler

Obtain the IP address of the Istio gateway run this command:

```bash
k8s describe svc istio-ingressgateway -n istio-system | grep IP
```

Which yields output:

```
IP:                       10.43.195.89
```

Typically the gateway listens on port 80 or 443. Do a quick check to see
if the traffic is spread across both instances:

```bash
curl 10.43.195.89/hello
```

Which after a few runs will yield one of these two lines:

```
Hello version: v1, instance: helloworld-v1-6757db4ff5-wjrfv
Hello version: v2, instance: helloworld-v2-85bc988875-hf4fn
```

If you can reach both, that means the autoscaler is sending traffic to both.
To benchmark it further, consider running some infinite loops of those curl
statements to simulate traffic.

```bash
[open a new terminal]
while true; do curl -s -o /dev/null "http://10.43.195.89/hello"; done
[open a second terminal]
while true; do curl -s -o /dev/null "http://10.43.195.89/hello"; done
```
