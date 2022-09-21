# Local Cluster Setup

Why on earth would we want to complicate things and use kubernetes locally when there are tools like docker and docker-compose? Well, we want to build systems that scale indefinitely and have the least amount of configuration possible.

## Requirements

- Helm V3
- Kubectl configured
- A kubernetes cluster

When you set up a kubernetes cluster you need a way to route traffic to the services you deploy into kubernetes. 

There are two options to do this, one way is the less than ideal and only useful really for debugging `port-forwarding` approach, the other is the more widely used and customizable ingress controller.

Ambassador is the ingress controller chosen, as it provides by far the best out-of-the-box local and remote development environment via the utility called `telepresence`. This is an extremely useful tool as you can develop locally and watch your changes happen real-time in a local or remote kubernetes cluster.

Ingress controllers like Traefik, Nginx, Kong, etc. lack this feature along with some smaller developer experience and quality of life features, so as such was chosen.

## Installation

[Go here and create an account](https://app.getambassador.io/cloud/welcome) if you have not already.

### Add the Helm Chart Repo

```bash
helm repo add datawire https://www.getambassador.io && \
helm repo update
```

### Install with Helm 3

Setup the namespace and install the custom kubernetes resource definitions.
```bash
kubectl create namespace ambassador && \
kubectl apply -f https://app.getambassador.io/yaml/edge-stack/latest/aes-crds.yaml
```

Now, you will need the installation cloud connect token the UI will give you for this command. This is where we depart from the usual installation steps since we want to have a config file to show the actual state of our system inside source control. 

Reason being, the default listeners and service only open up ports for HTTP1 and HTTP2, and again only the web ports. For local development we want tp actually use a wider range of services such as a cache and database engine, so the template file allows easy access and changing. So, to install, do this as you can customize the actual stuff happening inside the chart.

Everytime you want to open a port or protocol, simply put it in there and run this command.
```bash
helm upgrade -i edge-stack --namespace ambassador datawire/edge-stack --values ambassador.yaml
```

Wait for the deployment to finish.
```bash
kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes
```