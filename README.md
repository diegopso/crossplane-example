## Pre-requisties

- A `kubectl`, and `helm`.

## Provision resources from local

Add the private key to `foundation/tls.pem`, then:

```bash
cp /path/to/my-key/tls.pem ./foundation
helm dependencies build ./foundation
helm upgrade --install platform-foundation ./foundation --namespace platform-foundation --create-namespace --wait
```

### Uninstall

```bash
kubectl delete app/aws-eu1 -n platform-foundation # delete this first to avoid race condition
helm uninstall platform-foundation --namespace platform-foundation --wait
```

## View ArgoCD UI

```bash
kubectl get secret argocd-initial-admin-secret -n platform-foundation --template={{.data.password}} | base64 -d
kubectl port-forward svc/platform-foundation-argocd-server -n platform-foundation 8080:443
```

Use the secret printed and the user `admin` to see the [UI](https://localhost:8080/).

## Sealing required secrets with Kubeseal

```bash
kubectl create secret generic aws-secret --from-file=creds=./.secrets/aws-credentials.txt --dry-run=client -o yaml >> ./.secrets/aws-creds.yaml
cat ./.secrets/aws-creds.yaml | kubeseal --cert foundation/tls.crt -o yaml -n infrastructure > aws-eu1/crossplane/templates/aws-creds.yaml
```
