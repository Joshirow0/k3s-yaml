# k3s yaml files
This yaml files were built in:
- Arquitecture: arm64
- k3s version: v1.35.4+k3s1
- cilium: v1.19.1
For this repo, cilium raplace most of tools provided by default k3s.
This are the tools:
- storage: SQLite - isn't replaced
- proxy: Traefik -> replaced by cilium gateway api
- internal network policies: Flannel -> replaced by cilium
- loadbalancer: idk -> replaced by cilium

## Instalation
To install this cluster use:

curl -sfL https://get.k3s.io | sh -s - server --flannel-backend=none --disable-network-policy --disable=traefik

Then, you will need to install cilium. Install "Cilium CLI" from the official page:
https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/

Finally if you want to use gateway api, by default this is disable in k3s' system and uninstalled.
So, first you will need to install it from:
https://gateway-api.sigs.k8s.io/guides/getting-started/#install-standard-channel

**Warning: Be sure that the version you install it's compatible with your cilium version

Then enable it with the following commands:



