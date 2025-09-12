# istio-k8-demo
Kubernetes project demonstrating Istio service mesh

This repo documents how to deploy the **Bookinfo sample app** on Kubernetes, expose it with **NGINX Ingress**, and then add **Istio** to demonstrate service mesh features (traffic routing, canary releases, mTLS, etc.).

---

## 📦 Prerequisites

- Docker Desktop with Kubernetes enabled (or any K8s cluster)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured
- [Helm](https://helm.sh/docs/intro/install/)
- (Optional) [istioctl](https://istio.io/latest/docs/setup/getting-started/#download)

---

## 1. Deploy Bookinfo (without Istio)

Create namespace and deploy the app:

```bash
kubectl create ns bookinfo

kubectl apply -n bookinfo -f https://raw.githubusercontent.com/istio/istio/release-1.27/samples/bookinfo/platform/kube/bookinfo.yaml

kubectl -n bookinfo get pods
```

Deploy ingress for the app and verify

```bash
Kubectl apply -f ingress.yaml
kubectl -n bookinfo describe ingress bookinfo
```

## 2. Install Istio Control Plane (with Helm)

```bash
# add charts
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

# control-plane namespace
kubectl create ns istio-system

# CRDs/base
helm install istio-base istio/base -n istio-system \
  --set defaultRevision=default

# istiod (the mesh control plane)
helm install istiod istio/istiod -n istio-system --wait
```

## 3. Enable Istio Sidecar Injection

Label the app namespace and restart pods to get Envoy sidecars

```bash
kubectl label ns bookinfo istio-injection=enabled --overwrite

# restart app pods
kubectl -n bookinfo rollout restart deploy
kubectl -n bookinfo get pods -w
```

## 4. Configure DestinationRules

Define version subsets for traffic routing:

```bash
kubectl apply -n bookinfo -f destination-rules.yaml

kubectl -n bookinfo get destinationrules
```

## 5. Apply VirtualService Rules

a) Pin all traffic to v3 (red stars)

```bash
kubect apply -f vs-complete-traffic-to-single-version.yaml
```

b) Canary rollout (80% v1, 20% v3)

```bash
kubect apply -f vs-traffic-splitting.yaml
```

c) User-based routing (user praveen → v2)

```bash
kubect apply -f vs-user-based.yaml
```

d) Fault injection (user tester → 5s delay on v2)

```bash
kubect apply -f vs-fault-injection.yaml
```