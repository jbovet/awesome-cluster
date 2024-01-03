# Deployment Awesome Cluster

1. Fetch and save the kubeseal certificate from cluster in deploys/sealed-secrets/awesome-cluster-cert.pem
 `kubeseal --fetch-cert > deploys/sealed-secrets/awesome-cluster-cert.pem`
2. Create a kubernetes secret in cert-manager namespace with the cloudflare Global API Key and save to file cloudflare-api-key.yaml
`kubectl create secret generic cloudflare-api-key --from-literal=api-key=my-global-api-key --dry-run=client -n cert-manager -o yaml > cloudflare-api-key.yaml`
3. Create sealed secret with public sealed secret cert previously fetched
`kubeseal --format=yaml --cert=deploys/sealed-secrets/awesome-cluster-cert.pem < cloudflare-api-key.yaml > deploys/cert-manager/sealed-secret.yaml`
4. Commit, push and reconcile
    `flux reconcile source git flux-system`
5. Check if the secret `cloudflare-api-key` is decypted in the sealed secret controller at kube-system namespace
`kubectl -n kube-system logs -f sealed-secrets-controller-xxxxx-xxx`
you would see something like this
```Event(v1.ObjectReference{Kind:"SealedSecret", Namespace:"cert-manager", Name:"cloudflare-api-key", UID:"9bc5398f-dcc8-4822-b9bd-24b262a62a1f", APIVersion:"bitnami.com/v1alpha1", ResourceVersion:"7741", FieldPath:""}): type: 'Normal' reason: 'Unsealed' SealedSecret unsealed successfully update suppressed, no changes in sealed secret spec of cert-manager/cloudflare-api-key```
6. Check if certmanager create de dns records using the decrypted cloudflare-api-key
`kubectl -n cert-manager logs -f cert-manager-xxxxxx`