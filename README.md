# Homelab Infrastructure Manifest Files

## Clusters

### Staging    
- Talos VMs in vSphere
- Managed by Terraform
- Managed by FluxCD

### Production 
- Talos Nodes on Dell 3070s
- Manually booted into ISOs
- Managed by FluxCD

Talos install includes Cilium
Manual install of fluxcd

FluxCD installs Rook, nginx-ingress, cert-manager


brew install flux

make PAT token in Github

flux bootstrap github \
  --cluster-domain prod.mehar.internal \
  --token-auth \
  --owner=aaronmehar \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production

