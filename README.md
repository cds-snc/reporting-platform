# Reporting Platform

This is platform-as-code for the Report a cybercrime project. This is little
more than a hello world so far, but it's a start.

## Key ideas

We are using the `kustomize` tool which is now built into `kubectl`. This tool
to generate variations on our configuration and sorts the output so that
everything is created in the right order.

# Deployment

You can deploy this application with the following command:

```sh
# Bring it up in Minikube
kubectl kustomize manifests/overlays/minikube/ | kubectl apply -f -

# Bring it up on Azure
kubectl kustomize manifests/overlays/aks/ | kubectl apply -f -
```
