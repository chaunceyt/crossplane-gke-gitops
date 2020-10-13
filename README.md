# Manage GKE clusters using Crossplane with Fluxcd

Create small cluster. 

Kind cluster

`kind create cluster`


GKE cluster

```
gcloud beta container --project "[PROJECT_NAME]" clusters create "crossplane-manager" \
	--zone "us-central1-c" \
	--machine-type "n1-standard-4" \
	--num-nodes "1" \
	--enable-ip-alias
```


Install crossplane and provider

Good [documentation](https://crossplane.io/docs/v0.13/cloud-providers/gcp/gcp-provider.html)

```
kubectl create namespace crossplane-system
helm repo add crossplane-alpha https://charts.crossplane.io/alpha
helm install crossplane --namespace crossplane-system crossplane-alpha/crossplane --set alpha.oam.enabled=true

curl -sL https://raw.githubusercontent.com/crossplane/crossplane/release-0.13/install.sh | sh
chmod +x kubectl-crossplane
mv kubectl-crossplane $HOME/bin
kubectl crossplane install provider crossplane/provider-gcp:v0.12.0

# create gcp serviceaccount  https://crossplane.io/docs/v0.13/getting-started/install-configure.html#get-gcp-account-keyfile
kubectl create secret generic gcp-creds -n crossplane-system --from-file=key=./creds.json --dry-run=client -o yaml > gcp-creds-secret.yaml

# Install sealed-secrets

helm fetch stable/sealed-secrets --untar
helm template --name sealed-secrets-controller --namespace kube-system sealed-secrets/ | kubectl -n kube-system apply -f -
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.12.5/kubeseal-darwin-amd64
mv kubeseal-darwin-amd64 $HOME/bin/kubeseal
chmod +x $HOME/bin/kubeseal

cat gcp-creds-secret.yaml | kubeseal > gcp-creds-sealed-secret.yaml
kubectl apply -f gcp-creds-sealed-secret.yaml

# Install fluxcd

Create git repo and prepare to create `deploy keys`
Download this [flux-init.sh](https://raw.githubusercontent.com/swade1987/gitops-with-kustomize/master/scripts/flux-init.sh) script, then edit `REPO_GIT_INIT_PATHS` and `REPO_URL`
`./flux-init.sh`

```

## Other resources

- [crossplane github](https://github.com/crossplane)
- https://github.com/crossplane/crosscd
- [crossplane youtube](https://www.youtube.com/channel/UC19FgzMBMqBro361HbE46Fw)
- [fluxcd](https://github.com/fluxcd/flux)
- [flux-init.sh](https://github.com/swade1987/gitops-with-kustomize/blob/master/scripts/flux-init.sh) install script.

