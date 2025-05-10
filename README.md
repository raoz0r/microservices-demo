# Sock Shop Deployment (Deprecated)

> ⚠️ This project is no longer being maintained. The Sock Shop microservices demo, once useful for learning cloud-native development, has become outdated due to deprecated Kubernetes APIs and old Helm charts. This documentation is saved here for learning and reference. Development has moved to the [Google Cloud Platform Microservices Demo](https://github.com/GoogleCloudPlatform/microservices-demo).

## Summary

This document explains how the Sock Shop demo was deployed using Argo CD, Helm, and K3s. It walks through the setup process, the problems encountered, and how those problems were handled. Even though the deployment is no longer active, this write-up offers valuable lessons for students and developers working with Kubernetes and continuous delivery tools.

## Project Status

* ❌ **Development**: Stopped
* 🔍 **Why Deprecated**: Kubernetes 1.22 and later no longer support some of the APIs used by this demo. Also, its Helm charts are not actively updated.
* ✅ **Use Instead**: [`GoogleCloudPlatform/microservices-demo`](https://github.com/GoogleCloudPlatform/microservices-demo)

---

## System Overview

* **Kubernetes**: K3s distribution
* **Deployment Tool**: Argo CD
* **Templating**: Helm 3
* **Chart Path**: `deploy/kubernetes/helm-chart`
* **Namespace**: `sock-shop`

---

## Deployment Steps

### Argo CD Configuration

**Git Repository:**

* URL: `https://github.com/raoz0r/microservices-demo.git`
* Branch: `HEAD`
* Chart Path: `deploy/kubernetes/helm-chart`

**Kubernetes Cluster:**

* API URL: `https://kubernetes.default.svc`
* Namespace: `sock-shop`

**Helm Values:**

* File: `values.yaml`
* Key settings:

  * `istio.enabled` set to `false`
  * `ingress.annotations`: set to use `nginx`; hostname and TLS fields left blank
  * Resource limits and JVM settings adjusted for each container

**Sync Settings in Argo CD:**

* ✅ Prune resources before reapplying
* ✅ Only apply changes when out of sync
* ❌ Do not automatically create the namespace (create it manually first)

---

## Common Issues and Fixes

### 1. Invalid Helm Release Name

**Error:**
`invalid release name "sock shop"`

**Solution:**
Changed the app name to `sock-shop` to match naming rules (no spaces, lowercase, max 53 characters).

---

### 2. Missing Helm Dependencies

**Error:**
`found in Chart.yaml, but missing in charts/: nginx-ingress`

**Fix:**

```bash
helm dependency build
```

This command downloaded missing charts and fixed the error.

---

### 3. Deprecated Kubernetes CRD Version

**Error:**

```yaml
The Kubernetes API could not find version "v1beta1" of apiextensions.k8s.io/CustomResourceDefinition
```

**Cause:**
The chart used an old API version (`v1beta1`) that Kubernetes stopped supporting in version 1.22.

**Reference:**

* [https://kubernetes.io/docs/reference/using-api/deprecation-guide/#customresourcedefinition-v122](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#customresourcedefinition-v122)

**Outcome:**
Instead of upgrading the entire chart to the newer API version (`v1`), the project was dropped in favor of a more modern and maintained alternative.

---

## Clean-up Instructions

To fully remove all components of this deployment:

```bash
# Delete project files
rm -rf microservices-demo

# Clear Helm cache
rm -rf ~/.cache/helm/repository/nginx-ingress*

# Delete the namespace from Kubernetes
kubectl delete namespace sock-shop

# Check that it's gone
kubectl get ns
kubectl get all --all-namespaces | grep sock

# Remove the app from Argo CD
argocd app list
argocd app delete sock-shop --cascade
```

---

## License

This repository is for learning and documentation purposes. No license restrictions apply.
