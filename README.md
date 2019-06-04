# Reporting Platform

This is platform-as-code for the Report a cybercrime project. This is little
more than a hello world so far, but it's a start.

## Key ideas

We are using
[kustomize](https://mikewilliamson.wordpress.com/2019/06/01/kubernetes-config-with-kustomize/)
to generate variations on our configuration and sorts the output so that
everything is created in the right order.

Secrets are encrypted using [Sealed
Secrets](https://github.com/bitnami-labs/sealed-secrets), and then checked into
source control along with everything else. The Sealed Secrets controller lives
inside the cluster and decrypts the secrets. The Sealed Secrets controller
expects a standard secret called `sealed-secrets-key` to exist with decryption
keys for it to use.

## The basics

Creating sealed secrets requires the program kubeseal.
You can install it on a mac with

```bash
brew install kubeseal
```

Or, assuming a working golang environment with `go get`:

```
go get github.com/bitnami-labs/sealed-secrets/cmd/kubeseal
```

Next up, you need to create some keys with which to encrypt/decrypt.

```sh
# generate keys if you need to. Keep these somewhere safe
openssl req -x509 -nodes -days 365 -newkey rsa:4096 \
  -keyout private.key -out public.crt \
  -subj "/CN=report-a-cybercrime"
```

The public key is literally public and can be commited along side the code in
the repo.

The private key will need to added to the cluster to allow the secrets to be
decrypted:

```bash
kubectl create secret tls sealed-secrets-key \
  --key=private.key --cert=public.crt
  --namespace=kube-system
```

After that, store the private key somewhere safe.

## Creating a new sealed secret

Now anyone with the public key can encrypt secrets:

```bash
kubectl create secret generic database-secrets \
  --from-literal=DB_USER=somesecret --dry-run -o yaml \
 | kubeseal --cert=public.crt --format yaml - > database-secrets.yaml
```

## Deployment

You can deploy this application with the following command:

```sh
# assuming the sealed-secrets-key already exists in the cluster...

# Bring the app (with any encrypted secrets) up in Minikube
kubectl apply -k manifests/overlays/minikube

# Bring the app (with any encrypted secrets) up on Azure
kubectl apply -k manifests/overlays/aks
```
