# kube-dnsmasq

## Example with `build-ipxe`

Create a kustomization:

`kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- github.com/bzub/kube-dnsmasq
- github.com/bzub/kube-dnsmasq/build-ipxe
commonLabels:
  app.kubernetes.io/instance: my-dnsmasq

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

- name: dnsmasq
  behavior: merge
  literals:
  - |-
    dnsmasq.conf=
    no-daemon
    enable-tftp
    tftp-root=/var/lib/tftpboot

    # Legacy PXE
    dhcp-match=set:bios,option:client-arch,0
    dhcp-boot=tag:bios,undionly.kpxe

    # UEFI
    dhcp-match=set:efi32,option:client-arch,6
    dhcp-boot=tag:efi32,custom/ipxe.efi
    dhcp-match=set:efibc,option:client-arch,7
    dhcp-boot=tag:efibc,custom/ipxe.efi
    dhcp-match=set:efi64,option:client-arch,9
    dhcp-boot=tag:efi64,custom/ipxe.efi

    # iPXE
    dhcp-userclass=set:ipxe,iPXE
    dhcp-boot=tag:ipxe,http://assets.management.example.com:8081/boot.ipxe

    log-queries
    log-dhcp

patches:
- target:
    kind: Deployment
    name: dnsmasq
  patch: |-
    - op: add
      path: /spec/template/spec/initContainers/0/volumeMounts/-
      value:
        name: ipxe-dot-efi-dot-gz
        mountPath: /assets/ipxe.efi.gz
        subPath: ipxe.efi.gz
    - op: add
      path: /spec/template/spec/volumes/-
      value:
        name: ipxe-dot-efi-dot-gz
        configMap:
          name: ipxe-dot-efi-dot-gz
          items:
          - key: ipxe.efi.gz
            path: ipxe.efi.gz
```

Build and deploy:

```sh
kustomize build | kubectl apply -f -
```
