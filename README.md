# Headscale WireGuard Mesh VPN (`keksi.si`)

An enterprise-grade, self-hosted [Headscale](https://github.com/juanfont/headscale) control plane implementation (Tailscale-compatible WireGuard mesh network) configured for the domain **`keksi.si`** with **Port 53 UDP** STUN and DERP NAT traversal for firewall and captive-portal bypass.

---

## 🚀 Key Features

- **WireGuard Mesh Network**: Fully end-to-end encrypted direct peer-to-peer tunnels using WireGuard.
- **Domain & MagicDNS**: Integrated MagicDNS under base domain **`vpn.keksi.si`** (nodes reachable as `<node>.vpn.keksi.si`).
- **Firewall Bypass**: Embedded DERP & STUN relay listening on **UDP Port 3478** (forward external UDP port 53 -> 3478 on router for firewall bypass).
- **Headscale Web UI**: Clean dashboard at `https://headscale.keksi.si/web` for managing users, nodes, pre-authentication keys, and routing rules.
- **Helm & GitOps Ready**: Fully packaged **Helm Chart** (`chart/`) with ArgoCD GitOps integration and Longhorn persistent storage.
- **Dual Deployment Options**: Deploy via Helm / ArgoCD or standalone Docker Compose.

---

## 📁 Repository Structure

```
WireGuard/
├── argocd-app.yaml               # Root ArgoCD Application (Helm source)
├── chart/                        # Headscale Helm Chart
│   ├── Chart.yaml                # Helm chart metadata
│   ├── values.yaml               # Configurable Helm values (domain, ports, PVC, ingress)
│   └── templates/                # Kubernetes templates
│       ├── _helpers.tpl
│       ├── configmap.yaml
│       ├── pvc.yaml
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
├── k3s/                          # Raw Kubernetes manifests (Kustomize fallback)
├── docker-compose/               # Standalone Docker Compose Deployment
│   ├── docker-compose.yml
│   ├── config/
│   │   ├── config.yaml
│   │   └── acl.hujson
│   └── .env.example
└── README.md
```

---

## ⛵ Helm Deployment

### 1. Deploy via ArgoCD
Apply the ArgoCD Application manifest:
```bash
kubectl apply -f argocd-app.yaml
```

### 2. Deploy via Helm CLI Directly
```bash
helm upgrade --install headscale ./chart \
  --namespace headscale \
  --create-namespace
```

### 3. Customizing Values
You can override any value from `chart/values.yaml`:
```bash
helm upgrade --install headscale ./chart \
  --namespace headscale \
  --set headscale.serverUrl="https://headscale.keksi.si" \
  --set ingress.host="headscale.keksi.si"
```

---

## 🐳 Docker Compose Deployment (Standalone)

To run Headscale on a standalone VM or server:

```bash
cd docker-compose
cp .env.example .env
docker compose up -d
```

---

## 🔑 Administrative Tasks (Headscale CLI)

Run these commands inside the `headscale` container:

### Generate API Key for Headscale Web UI
```bash
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale apikeys create
```
Paste this API Key into the Web UI dashboard at `https://headscale.keksi.si/web/settings.html` to link the UI.

### Create a User
```bash
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale users create admin
```

### Generate a Pre-Auth Key (Reusable or Single-Use)
```bash
# Generate a reusable key valid for 90 days
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale preauthkeys create -u admin --reusable --expiration 90d
```

### List Connected Nodes
```bash
kubectl exec -it deployment/headscale -n headscale -c headscale -- headscale nodes list
```

---

## 💻 Client Setup Guide

### 🐧 Linux (Tailscale CLI)
```bash
sudo tailscale up --login-server=https://headscale.keksi.si --authkey=<YOUR_PREAUTH_KEY>
```

### 🪟 Windows & 🍎 macOS
1. Install the official Tailscale client.
2. In Terminal / PowerShell, run:
```powershell
tailscale.exe up --login-server=https://headscale.keksi.si
```

### 📱 iOS & Android
1. Install Tailscale from App Store / Google Play.
2. Tap the top-right 3 dots icon -> select **Custom login server**.
3. Enter `https://headscale.keksi.si`.
