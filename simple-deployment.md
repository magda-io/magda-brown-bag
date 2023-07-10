# Simply Deployment with published Helm Chart

1> Install kubernetes-replicator

> It’s only required by the OpenFaas part of Magda which can be turned off via [global.openfaas.enabled](https://github.com/magda-io/magda/tree/master/deploy/helm/magda).

```bash
# add helm chart repo
helm repo add mittwald https://helm.mittwald.de

# update helm chart repo
helm repo update

# create namespace `kubernetes-replicator`
kubectl create namespace kubernetes-replicator

# Install kubernetes-replicator via helm
helm upgrade --namespace kubernetes-replicator --install kubernetes-replicator mittwald/kubernetes-replicator
```

2> Create a namespace "magda" for your Magda installation

```bash
kubectl create namespace magda
```

3> Create required secrets (optional)

# Optional; Required by correspondence-api --- for sending email notification of user inquires
kubectl create secret generic smtp-secret --namespace magda --from-literal=username=$SMTP_USERNAME --from-literal=password=$SMTP_PASSWORD
```

4> Install Magda via Helm

```bash
helm upgrade --namespace magda --install --timeout 9999s --set magda-core.gateway.service.type=LoadBalancer magda oci://ghcr.io/magda-io/charts/magda
```

> By default, Helm will install the latest production version of Magda. You can use `--version` to specify the exact chart version to use. e.g.:

```bash
helm upgrade --namespace magda --install --version 2.2.4 --timeout 9999s --set magda-core.gateway.service.type=LoadBalancer magda oci://ghcr.io/magda-io/charts/magda
```

The value `--set magda-core.gateway.service.type=LoadBalancer` will expose Magda via load balancer.

You can run:

```bash
echo $(kubectl get svc --namespace magda gateway --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
```

to find out the load balancer external IP. And access Magda via http://[External IP].

> If you use minikube, you will need to run [minikube tunnel](https://minikube.sigs.k8s.io/docs/handbook/accessing/#loadbalancer-access) to get external IP.

> You can also follow [this doc](https://github.com/magda-io/magda/blob/master/docs/docs/how-to-setup-https-to-local-cluster.md) to setup local HTTPS access to Magda
