# k8s-doom

Doom 2 multiplayer server running on Kubernetes. Uses the Zandronum source port with Freedoom2 (open-source Doom 2 replacement WAD). Deploys to any Kubernetes cluster with an init container that automatically downloads the WAD — no manual file management needed.

## Quick Start

```bash
git clone https://github.com/ghost1y-sh/k8s-doom.git
cd k8s-doom/doom-server
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/server.yaml
```

Get the server address:

```bash
kubectl get svc -n doom
```

Connect with Zandronum client to the `EXTERNAL-IP` on port `10666` UDP.

## Architecture

```
Internet (UDP 10666)
  └── Network Load Balancer
        └── doom-service
              └── doom-server pod
                    ├── init: alpine (downloads freedoom2.wad)
                    └── main: zandronum-server (official-latest)
```

The init container downloads the Freedoom2 WAD automatically on pod startup. The ConfigMap stores all server configuration — game mode, maps, frag limit, skill level — editable without rebuilding anything.

## Configuration

All settings are in the ConfigMap inside `k8s/server.yaml`. Edit directly:

```bash
kubectl edit configmap doom-config -n doom
kubectl rollout restart deployment doom-server -n doom
```

### Game Modes

Uncomment one in the ConfigMap:

| Mode | Setting | Description |
|------|---------|-------------|
| Deathmatch | `set deathmatch 1` | Free-for-all |
| Cooperative | `set cooperative 1` | Players vs monsters |
| Team Play | `set teamplay 1` | Team deathmatch |
| CTF | `set ctf 1` | Capture the flag |

### Maps

Uncomment maps in the ConfigMap to add them to rotation. All 32 Doom 2 maps are listed:

```
addmap MAP07    // Dead Simple (active)
addmap MAP08    // Tricks and Traps (active)
// addmap MAP01    // Entryway (uncomment to add)
// addmap MAP02    // Underhalls
...
```

### Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `sv_maxplayers` | 8 | Maximum players |
| `fraglimit` | 30 | Kills to win |
| `skill` | 3 | 1=Easy, 3=Medium, 5=Nightmare |
| `sv_forcepassword` | disabled | Set a password to make private |

## Using Your Own WAD

To use the original `DOOM2.WAD` instead of Freedoom2, replace the init container with a persistent volume containing your WAD file, and change the `-iwad` argument to `/data/DOOM2.WAD`. You must own a legitimate copy of Doom 2.

## Connecting

### Install Client

Download Zandronum from [zandronum.com/download](https://zandronum.com/download). Players also need the Freedoom2 WAD from [freedoom.github.io](https://freedoom.github.io/).

### Direct Connect

In Doomseeker (bundled with Zandronum): File → Quick Connect → enter your server's `EXTERNAL-IP` and port `10666`.

### Master Server

If `sv_updatemaster` is set to `1`, the server appears on the Zandronum master server list. Players can find it by searching for the server name.

### Command Line

```bash
zandronum -connect your-server-address:10666
```

## Management

```bash
# Check status
kubectl get pods -n doom
kubectl logs -n doom deployment/doom-server

# Restart server
kubectl rollout restart deployment doom-server -n doom

# Stop server
kubectl delete -f k8s/server.yaml

# Start server
kubectl apply -f k8s/server.yaml

# Full teardown
kubectl delete namespace doom
```

## Cloud Considerations

The service is configured for AWS EKS with a Network Load Balancer (NLB) via annotations. For other cloud providers or bare metal, change the Service type:

**Bare metal / on-prem:**
```yaml
spec:
  type: NodePort
  ports:
    - port: 10666
      targetPort: 10666
      nodePort: 30666
      protocol: UDP
```

**GKE:** Remove the AWS annotations — GKE creates a TCP/UDP load balancer automatically with `type: LoadBalancer`.

## Requirements

- Kubernetes cluster (EKS, GKE, bare metal, etc.)
- kubectl configured
- For AWS EKS: AWS Load Balancer Controller installed
- Players: Zandronum client + Freedoom2 WAD

## License

MIT
