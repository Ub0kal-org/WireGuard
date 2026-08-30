# Headscale WireGuard Mesh VPN (`keksi.si`)

An enterprise-grade, self-hosted [Headscale](https://github.com/juanfont/headscale) control plane implementation (Tailscale-compatible WireGuard mesh network) configured for the domain **`keksi.si`** with **Port 53 UDP** STUN and DERP NAT traversal for firewall and captive-portal bypass.

---

## 🚀 Key Features

- **WireGuard Mesh Network**: Fully end-to-end encrypted direct peer-to-peer tunnels using WireGuard.
- **Domain & MagicDNS**: Integrated MagicDNS under base domain **`vpn.keksi.si`** (nodes reachable as `<node>.vpn.keksi.si`).
- **Firewall Bypass**: Embedded DERP & STUN relay listening on **UDP Port 3478** (forward external UDP port 53 -> 3478 on router for firewall bypass).
- **Headscale Web UI**: Clean dashboard at `https://headscale.keksi.si/web` for managing users, nodes, pre-authentication keys, and routing rules.
- **Automated Bootstrap Token Generation**: Automated Helm Hook Job initializes the `admin` user, generates a 365-day API token, and securely stores it in a Kubernetes Secret (`headscale-api-credentials`).
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
│       ├── ingress.yaml
│       ├── rbac.yaml             # Bootstrap RBAC
│       └── bootstrap-job.yaml    # Automated user & API token generator
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

---

## 🔑 Retrieving the Automated API Key

The Helm chart automatically generates the API key during deployment. You can retrieve it anytime:

```bash
# Retrieve the generated API Key from the secret:
kubectl get secret headscale-api-credentials -n headscale -o jsonpath='{.data.api-key}' | base64 -d
echo ""
```

### ⚡ 1-Click Browser Login into the Web UI
Open `https://headscale.keksi.si/web`, press **F12** (Open Console), and paste:

```javascript
localStorage.setItem('headscaleURL', 'https://headscale.keksi.si');
localStorage.setItem('headscaleAPIKey', '<YOUR_API_KEY>');
location.href = '/web';
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
