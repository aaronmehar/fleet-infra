
## Clear old files
```bash
rm -f controlplane.yaml worker.yaml talosconfig kubeconfig secrets.yaml
```


## Create Control-panel node config overrides
```yaml
cat << EOF > patches/cp-patch.yaml
cluster:
  etcd:
    advertisedSubnets:
      - 172.29.232.0/24
machine:
  network:
    interfaces:
    - interface: enp1s0
      dhcp: true
      vip:
        ip: 172.29.232.40
EOF
```
## Create all node config overrides
```yaml
cat << EOF > patches/all-patches.yaml
machine:
  install:
    disk: /dev/nvme0n1
  kubelet:
    nodeIP:
      validSubnets:
        - 172.29.232.0/24
  systemDiskEncryption:
    ephemeral:
      provider: luks2
      keys:
        - slot: 0
          tpm: {}
    state:
      provider: luks2
      keys:
        - slot: 0
          tpm: {}
cluster:
  allowSchedulingOnControlPlanes: true
  network:
    cni:
      name: custom
      urls:
        - http://172.29.232.2/k8s/prod/cilium.yaml
    dnsDomain: prod.mehar.internal
    podSubnets:
      - 100.96.0.0/11
    serviceSubnets:
      - 100.80.0.0/15
  proxy:
    disabled: true
EOF
```

## Configure LLDP as bootstrap fails until this is added
```yaml
cat << EOF > patches/configure-lldpd.yaml
---
apiVersion: v1alpha1
kind: ExtensionServiceConfig
name: lldpd
configFiles:
  - content: |
      configure lldp portidsubtype ifname
      unconfigure lldp management-addresses-advertisements
      unconfigure lldp capabilities-advertisements
      configure system description "Talos Node"
    mountPath: /usr/local/etc/lldpd/lldpd.conf
EOF
```

## Generate secreat material
```bash
talosctl gen secrets -o secrets.yaml
```

## Generate configuration with overrides
```bash
talosctl gen config --with-secrets secrets.yaml k8s-prod https://k8s.prod.mehar.internal:6443 --install-image=factory.talos.dev/metal-installer-secureboot/ede89ce2a46f773b96875ab18e7c9afa3e5bc41cd21ec920e92bf85949a14f69:v1.10.4   --config-patch-control-plane @patches/cp-patch.yaml --config-patch @patches/all-patches.yaml  --config-patch @patches/configure-lldpd.yaml
```

## Apply configuration 
```bash
talosctl -n 172.29.232.41 apply-config --insecure -f controlplane.yaml
talosctl -n 172.29.232.42 apply-config --insecure -f controlplane.yaml
talosctl -n 172.29.232.43 apply-config --insecure -f controlplane.yaml
talosctl -n 172.29.232.44 apply-config --insecure -f worker.yaml
talosctl -n 172.29.232.45 apply-config --insecure -f worker.yaml
```

## Bootstrap Kubernetes
```bash
talosctl bootstrap --nodes 172.29.232.41 --endpoints 172.29.232.41  --talosconfig=./talosconfig
```

## Get Kubernetes client configuration
```bash
talosctl kubeconfig --nodes 172.29.232.41 --endpoints 172.29.232.41 --talosconfig=./talosconfig
```

## Useful Talos commands
```bash
talosctl --nodes 172.29.232.41 --endpoints 172.29.232.41 health --talosconfig=./talosconfig
talosctl --nodes 172.29.232.41 --endpoints 172.29.232.41 dashboard --talosconfig=./talosconfig
talosctl config endpoint 172.29.232.41 172.29.232.42 172.29.232.43  --talosconfig=./talosconfig
 ```


# Helm Template commands

## Cilium
```bash
helm template \
    cilium \
    cilium/cilium \
    --version 1.15.6 \
    --namespace cilium \
    --set ipam.mode=kubernetes \
    --set kubeProxyReplacement=true \
    --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
    --set cgroup.autoMount.enabled=false \
    --set cgroup.hostRoot=/sys/fs/cgroup \
    --set k8sServiceHost=localhost \
    --set k8sServicePort=7445 > /usr/share/nginx/html/k8s/prod/cilium.yaml
```