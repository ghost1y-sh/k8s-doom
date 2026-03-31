# k8s-doom

Doom 2 multiplayer server running on Kubernetes. Uses Zandronum source port with Freedoom2 WAD (open-source Doom 2 replacement).

## Deploy
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/server.yaml
kubectl get svc -n doom
```

Connect with Zandronum client to the service address on port 10666 UDP.

## Configuration

Edit the ConfigMap in `k8s/server.yaml` to change game mode, maps, frag limit, etc. Uncomment maps to add them to rotation.
```bash
kubectl edit configmap doom-config -n doom
kubectl rollout restart deployment doom-server -n doom
```

## Architecture

- Init container downloads Freedoom2 WAD automatically
- Zandronum server runs as main container
- NLB exposes UDP 10666 to the internet
- Registers with Zandronum master server for public listing
