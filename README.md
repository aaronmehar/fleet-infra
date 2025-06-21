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


