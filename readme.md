
> tested on rancher-desktop

### install linkerd
```sh
cd certs
# crds
helm repo add linkerd https://helm.linkerd.io/stable
helm install linkerd-crds linkerd/linkerd-crds \
  -n linkerd --create-namespace 
# control plane
helm upgrade -i install linkerd-control-plane \
  -n linkerd \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  linkerd/linkerd-control-plane
```
### install kyverno
```sh
helm repo add kyverno https://kyverno.github.io/kyverno/
cd ../kyverno
helm upgrade -i kyverno kyverno/kyverno -n kyverno --create-namespace -f values.yaml

```
### re-tag and sign linkerd images
```sh
# sign images or use existing certs
nerdctl pull cr.l5d.io/linkerd/proxy:stable-2.12.0
nerdctl pull cr.l5d.io/linkerd/proxy-init:v2.0.0
nerdctl tag cr.l5d.io/linkerd/proxy-init:v2.0.0 galsolom/proxy-init:v2.0.0
nerdctl tag cr.l5d.io/linkerd/proxy:stable-2.12.0 galsolom/proxy:stable-2.12.0
nerdctl push galsolom/proxy-init:v2.0.0
nerdctl push galsolom/proxy:stable-2.12.0
cosign generate-key-pair
cosign sign --key cosign.key galsolom/proxy-init:v2.0.0
cosign verify --key cosign.pub galsolom/proxy-init:v2.0.0
cosign sign --key cosign.key galsolom/proxy:stable-2.12.0
cosign verify --key cosign.pub galsolom/proxy:stable-2.12.0
```

### apply verify images policy
```sh
kubectl apply -f verifyimages.yaml
# wait few seconds for the webhook to be ready
```
```sh
# deployment should pass,but RS fails on one of the non-linkerd images, no pods are created.
kubectl apply -f many.yaml
```
this repo contains
* throw-away certs


zz