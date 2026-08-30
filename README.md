# Headscale WireGuard Mesh VPN (`keksi.si`)

An enterprise-grade, self-hosted [Headscale](https://github.com/juanfont/headscale) control plane implementation (Tailscale-compatible WireGuard mesh network) configured for the domain **`keksi.si`** with **Port 53 UDP** STUN and DERP NAT traversal for firewall and captive-portal bypass.

---

## 🚀 Key Features

- **WireGuard Mesh Network**: Fully end-to-end encrypted direct peer-to-peer tunnels using WireGuard.
- **Domain & MagicDNS**: Integrated MagicDNS under base domain **`keksi.si`** (nodes reachable as `nodename.keksi.si`).
- **Firewall Bypass on Port 53**: Embedded DERP & STUN relay listening on **UDP Port 53** (standard DNS port), allowing connections through restrictive hotel Wi-Fi, guest networks, and corporate firewalls.
- **Headscale Web UI**: Clean dashboard for managing users, nodes, pre-authentication keys, and routing rules.
- **GitOps Ready**: Preconfigured for K3s Kubernetes with ArgoCD and Longhorn persistent storage.
- **Dual Deployment Options**: Deploy via Kubernetes K3s (Kustomize/ArgoCD) or standalone Docker Compose.

---

## 📁 Repository Structure

```
WireGuard/
├── argocd-app.yaml               # Root ArgoCD Application manifest
├── k3s/                          # K3s Kubernetes Manifests (Kustomize)
│   ├── kustomization.yaml        # Kustomize index
│   ├── namespace.yaml            # 'headscale' namespace
│   ├── pvc.yaml                  # Longhorn PVC for DB and encryption keys
│   ├── configmap.yaml            # Headscale config.yaml and ACL definitions
│   ├── deployment.yaml           # Headscale server + Headscale UI containers
│   ├── service.yaml              # ClusterIP and UDP 53 STUN LoadBalancer
│   └── ingress.yaml              # Caddy Ingress for headscale.keksi.si
├── docker-compose/               # Standalone Docker Compose Deployment
│   ├── docker-compose.yml        # Docker Compose configuration
│   ├── config/
│   │   ├── config.yaml           # Headscale configuration
│   │   └── acl.hujson            # Default ACL rules
│   └── .env.example              # Environment variables template
└── README.md
```

---

## ☸️ K3s / ArgoCD Deployment

### 1. Deploy via ArgoCD
Apply the root application manifest to your cluster:
```bash
kubectl apply -f argocd-app.yaml
```

### 2. Manual Apply via Kustomize (Direct)
If deploying without ArgoCD:
```bash
kubectl apply -k k3s/
```

### 3. Verification
Verify all pods and services are running:
```bash
kubectl get pods,svc,ingress -n headscale
```

---

## 🐳 Docker Compose Deployment (Standalone)

To run Headscale on a standalone VM or server:

```bash
cd docker-compose
cp .env.example .env
docker compose up -d
```

Check status:
```bash
docker compose ps
docker compose logs -f headscale
```

---

## 🔑 Administrative Tasks (Headscale CLI)

Run these commands inside the `headscale` container (via `kubectl exec` or `docker exec`):

### Create a User
```bash
# K3s
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale users create admin

# Docker Compose
docker compose exec headscale headscale users create admin
```

### Generate a Pre-Auth Key (Reusable or Single-Use)
```bash
# Generate a reusable key valid for 90 days
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale preauthkeys create -u admin --reusable --expiration 90d
```

### List Nodes & Check Status
```bash
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale nodes list
```

### Generate API Key for Headscale Web UI
```bash
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale apikeys create
```
Paste this API Key into the Headscale Web UI dashboard at `https://headscale.keksi.si/web` to link the UI.

---

## 💻 Client Setup Guide

### 🐧 Linux (Tailscale CLI)
```bash
# 1. Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 2. Login to your Headscale server
sudo tailscale up --login-server=https://headscale.keksi.si --authkey=<YOUR_PREAUTH_KEY>
```

### 🪟 Windows & 🍎 macOS
1. Install official Tailscale client.
2. In Terminal / PowerShell, run:
```powershell
tailscale.exe up --login-server=https://headscale.keksi.si
```
3. Alternatively, on macOS / Windows holding **Alt / Option** when clicking the Tailscale menu bar icon to select **Custom Login Server** and enter:
   `https://headscale.keksi.si`

### 📱 iOS & Android
1. Install Tailscale from App Store / Google Play.
2. Tap the top-right 3 dots icon -> select **Custom login server**.
3. Enter `https://headscale.keksi.si`.
4. Log in using your registered user or pre-auth key.

---

## 🌐 Firewall Bypass via Port 53 UDP

Headscale includes an embedded STUN & DERP relay configured to listen on **UDP port 53**:
- Port 53 UDP (DNS) is virtually never blocked on public Wi-Fi, hotels, universities, or cellular networks.
- Tailscale clients automatically perform STUN discovery and establish WireGuard peer-to-peer tunnels using UDP 53 when standard WireGuard UDP ports (e.g. 41641 / 51820) are filtered.
