rm -f controlplane.yaml worker.yaml talosconfig kubeconfig

talosctl gen config --with-secrets secrets.yaml k8s-prod https://k8s.prod.mehar.internal:6443 --config-patch=@all-patch.yaml --config-patch-control-plane @cp-patch.yaml

export TALOSCONFIG=$PWD/talosconfig

# Apply configuration
talosctl apply-config --insecure -n 172.29.232.41 --file controlplane.yaml
talosctl apply-config --insecure -n 172.29.232.42 --file controlplane.yaml
talosctl apply-config --insecure -n 172.29.232.43 --file controlplane.yaml
talosctl apply-config --insecure -n 172.29.232.44 --file worker.yaml
talosctl apply-config --insecure -n 172.29.232.45 --file worker.yaml
