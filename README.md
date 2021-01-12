# Helm Chart for Akri ValidatingConfigurationWebhook for Configurations

1. Install Akri
1. Install [`cert-manager`](https://cert-manager.io)
1. Create Certificate
1. Install Chart
1. Verify
1. Delete

## Certificate

```bash
WEBHOOK="akri-webhook"
NAMESPACE="default"

echo "
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ${WEBHOOK}
  namespace: ${NAMESPACE}
spec:
  secretName: ${WEBHOOK}
  duration: 8760h
  renewBefore: 720h
  isCA: false
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  usages:
    - server auth
  dnsNames:
  - ${WEBHOOK}.${NAMESPACE}.svc
  - ${WEBHOOK}.${NAMESPACE}.svc.cluster.local
  issuerRef:
    name: ca
    kind: Issuer
    group: cert-manager.io
" | kubectl apply --filename=-
```

This generates a Secret called `${WEBHOOK}` in `${NAMESPACE}`:

```bash
kubectl get secret/${WEBHOOK} \
--namespace=${NAMESPACE}
```

## `helm install ...`

```bash
CABUNDLE=$(\
  kubectl get secret/${WEBHOOK} \
  --namespace=${NAMESPACE} \
  --output=jsonpath="{.data.ca\.crt}") && echo ${CABUNDLE}

helm install webhook ./akri-webhook-helm \
--namespace=${NAMESPACE} \
--set=webhook.caBundle=${CABUNDLE}
```

## Verify

```bash
kubectl get validatingwebhookconfiguration/${WEBHOOK} \
--namespace=${NAMESPACE}
```

## Delete

```bash
microk8s.helm3 uninstall webhook \
--namespace=${NAMESPACE}
```
