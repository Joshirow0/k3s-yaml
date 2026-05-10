# K3s Cluster - Infrastructure & Manifests (ARM64)

Welcome to the documentation for my personal server infrastructure. This repository contains the declarative manifests (YAML) used to deploy and manage a K3s cluster.

## Environment Specifications
- **Architecture:** `arm64`
- **K3s Version:** `v1.35.4+k3s1`
- **Cilium Version:** `v1.19.1`

##  Architecture & Component Substitutions

For this project, I decided to strip out most of the default tools bundled with K3s. Instead, the cluster relies on **Cilium** as the core engine for networking, security, and routing, providing a more robust environment.

Below is a breakdown of the architectural changes:

| Component | K3s Default | Replacement in this Cluster | Comments |
| :--- | :--- | :--- | :--- |
| **Storage (State)** | SQLite | *None (Kept SQLite)* | Lightweight and ideal for a single-node edge computing environment. |
| **Storage (Pods)** | LocalPath | *None (Kept LocalPath)* | Willing to implement Longhorn when needed in the future. |
| **CNI (Networking)** | Flannel | **Cilium** | eBPF-based routing, offering superior performance and security policies. |
| **Proxy / Ingress** | Traefik | **Cilium Gateway API** | Modern Kubernetes standard; native L4 to L7 routing directly from the CNI. |
| **LoadBalancer** | Klipper-lb | **Cilium L2 Announcements** | Allows assigning local network IPs directly to K8s services without relying on external load balancers. |

---

##  Scratch Installation Guide

If you need to recreate this cluster, follow these steps in order, as the network dependencies are critical.

### Step 1: Install K3s (Barebones Mode)

First, we install K3s but explicitly disable its default networking components (Flannel, Traefik, and Klipper-lb) to prevent port conflicts with Cilium.

```bash
curl -sfL https://get.k3s.io | sh -s - server \
  --flannel-backend=none \
  --disable-network-policy \
  --disable=traefik \
  --disable=servicelb
```
### Step 2: Install Gateway API CRDs

**Cilium Gateway API** requires the standard Kubernetes Gateway API Custom Resource Definitions (CRDs) to be present in the cluster *before* installing Cilium.

> ⚠️ **Compatibility Warning:** Ensure that the Gateway API version you install is fully compatible with your current Cilium version.

Check the [Gateway API] (https://gateway-api.sigs.k8s.io/guides/getting-started/#install-standard-channel) page for a complete installation.For this case, the installed version is v1.4.0 with the following command:

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/standard-install.yaml
```

### Step 3: Install & Configure Cilium

With the groundwork laid, proceed to install Cilium, enabling its advanced routing and local IP announcement features. 

*Make sure you have the [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli) installed on your machine before running this:*

```bash
cilium install \
  --set kubeProxyReplacement=true \
  --set gatewayAPI.enabled=true \
  --set l2announcements.enabled=true
```

Once installed, verify the status of cilium with:

```bash
cilium status --wait
```

### Step 4: Apply Base Network Policies

Finally, review each of the base manifests, personalize them for your needs (IP ranges, domains, email addresses), and apply them. Here is exactly what each file does:

| Manifest | Description | Priority |
| :--- | :--- | :--- |
| **cilium-IP-pool.yaml** | Provides an IP pool for your services | High |
| **cilium-gateway.yaml** | Defines the main gateway to access to your services | High (Isn't needed if Ingress is used) |
| **cert.yaml** | Declaration of your certificates | Medium (SSL/TLS) |
| **hubble-route.yaml** | Route to hubble UI | Low |

After personalice them, apply with:

```bash
kubectl apply -f cilium-IP-pool.yaml
kubectl apply -f cilium-gateway.yaml
kubectl apply -f cert.yaml
```

Then you can install the services that you need. In this repo you will find a dir with the manifest and a README.md for each service that I had deployed.

---