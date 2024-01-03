# Deployment Awesome Cluster

1.- Create a kubernetes secret in cert-manager namespace with the cloudflare token and save to file cloudflare-api-key.yaml

`kubectl create secret generic cloudflare-api-key --from-literal=api-key=my-secret-token --dry-run=client -n cert-manager -o yaml > cloudflare-api-key.yaml`

2.- Create sealed secret with public cert deploys/sealed-secrets/awesome-cluster-cert.pem 

`kubeseal --format=yaml --cert=deploys/sealed-secrets/awesome-cluster-cert.pem < cloudflare-api-key.yaml > sealed-cloudflare-api-key.yaml`

3.- Create or Replace the sealed secret with sealed-cloudflare-api-key.yaml content in file deploys/cert-manager/sealed-secret.yaml

4.- Commit and push

5.- Check if the secret `cloudflare-api-key` is decypted in the sealed secret controller at kube-system namespace

`kubectl -n kube-system logs -f sealed-secrets-controller-xxxxx-xxx`

6.- Check if certmanager create de dns records using the decrypted cloudflare-api-key

`kubectl -n cert-manager logs -f cert-manager-xxxxxx`

