# Deployment Awesome Cluster

* Fetch and save the kubeseal certificate from cluster in deploys/sealed-secrets/awesome-cluster-cert.pem
 `kubeseal --fetch-cert > deploys/sealed-secrets/awesome-cluster-cert.pem`

* Create a kubernetes secret in cert-manager namespace with the cloudflare Global API Key and save to file cloudflare-api-key.yaml
`kubectl create secret generic cloudflare-api-key --from-literal=api-key=my-global-api-key --dry-run=client -n cert-manager -o yaml > cloudflare-api-key.yaml`

* Create sealed secret with public sealed secret cert previously fetched
`kubeseal --format=yaml --cert=deploys/sealed-secrets/awesome-cluster-cert.pem < cloudflare-api-key.yaml > deploys/cert-manager/sealed-secret.yaml`

* Commit, push and reconcile
    `flux reconcile source git flux-system`

* Check if the secret `cloudflare-api-key` is decypted in the sealed secret controller at kube-system namespace
`kubectl -n kube-system logs -f sealed-secrets-controller-xxxxx-xxx`
you would see something like this
```Event(v1.ObjectReference{Kind:"SealedSecret", Namespace:"cert-manager", Name:"cloudflare-api-key", UID:"9bc5398f-dcc8-4822-b9bd-24b262a62a1f", APIVersion:"bitnami.com/v1alpha1", ResourceVersion:"7741", FieldPath:""}): type: 'Normal' reason: 'Unsealed' SealedSecret unsealed successfully update suppressed, no changes in sealed secret spec of cert-manager/cloudflare-api-key```

* Check if certmanager create de dns records using the decrypted cloudflare-api-key
`kubectl -n cert-manager logs -f cert-manager-xxxxxx`
you will see something like this, wait a couple of minutes to finish propagation.
```cert-manager/challenges: propagation check failed" err="DNS record for \"hello-crab.awesome-cluster.jpbd.dev\" not yet propagated" resource_name="hello-crab-tls-1-1512669330-744811377" resource_namespace="hello-crab" resource_kind="Challenge" resource_version="v1" dnsName="hello-crab.awesome-cluster.jpbd.dev" type="DNS-01``` 
after that, you will see
```"cert-manager/certificaterequests-issuer-acme/sign: certificate issued" resource_name="hello-crab-tls-1" resource_namespace="hello-crab" resource_kind="CertificateRequest" resource_version="v1" related_resource_name="hello-crab-tls-1-1512669330" related_resource_namespace="hello-crab" related_resource_kind="Order" related_resource_version="v1"I0103 02:05:15.849262       1 conditions.go:252] Found status change for CertificateRequest "hello-crab-tls-1" condition "Ready": "False" -> "True"; setting lastTransitionTime to 2024-01-03 02:05:15.849249067 +0000 UTC m=+262.914682800```

Now you can check visit the site https://hello-crab.awesome-cluster.jpbd.dev/