# build-ipxe

## Example

Create a kustomization:

`kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- github.com/bzub/kube-dnsmasq/build-ipxe
commonLabels:
  app.kubernetes.io/instance: my-ipxe

configMapGenerator:
- name: build-ipxe
  behavior: merge
  literals:
  - MAKE_ARGS=bin-x86_64-efi/ipxe.efi
  - |-
    EMBED_SCRIPT=
    #!ipxe
    dhcp
    chain http://assets.management.example.com:8081/boot.ipxe
```

Build and deploy:

```sh
kustomize build | kubectl apply -f -
```

A ConfigMap called `ipxe-dot-efi-dot-gz` will be created containing a gzipped
customized build of `ipxe.efi`.
