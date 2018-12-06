# Kubernetes Metadata injection for New Relic APM agents

## How does it work ?

New Relic requires the following environment variables to identify Kubernetes objects in the APM agents:
- `K8S_CLUSTER_NAME`
- `K8S_NODE_NAME`
- `K8S_NAMESPACE_NAME`
- `K8S_DEPLOYMENT_NAME`
- `K8S_POD_NAME`
- `K8S_CONTAINER_NAME`

These environment variables can be set manually by the customer, or they can be automatically injected using a MutatingAdmissionWebhook.
New Relic provides an easy method for deploying this automatic approach.

## Automatic environment variable injection

### 1) Check if MutatingAdmissionWebhook is enabled on your cluster

This feature requires Kubernetes 1.9 or later. Verify that the kube-apiserver process has the admission-control flag set.

```
$ kubectl api-versions | grep admissionregistration.k8s.io/v1beta1
admissionregistration.k8s.io/v1beta1
```

### 2) Install the injection

```
$ kubectl apply -f newrelic-metadata-injection.yaml
```

Executing this
- creates `newrelic-metadata-injection-deployment` and `newrelic-metadata-injection-svc`,
- registers the `newrelic-metadata-injection-svc` service as a MutatingAdmissionWebhook with the Kubernetes api

### 3) Enable the automatic Kubernetes metadata injection on your namespaces

The injection is only applied to namespaces that have the `newrelic-metadata-injection` label set to `enabled`.

```
$ kubectl label namespace <namespace> newrelic-metadata-injection=enabled
```

## Prototype

This repo contains a prototype based on https://github.com/morvencao/kube-mutating-webhook-tutorial/.

### Build

The prototype uses dep as the dependency management tool:

```
go get -u github.com/golang/dep/cmd/dep
```

Build and push the docker image:

```
./build.sh
```

### TODO

- Currently the caBundle for the MutatingAdmissionWebhook has to be created by a script (`./create-certs.sh`)
- Istio has a fully automated way: they calculate the caBundle in the service and register it with the K8s api (this requires a Service Account)
