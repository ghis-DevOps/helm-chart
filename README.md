# Kubernetes Helm Chart and Examples

This repository contains the `app1` Helm chart and standalone Kubernetes manifests for practicing common cluster resources and configurations.

## Requirements

- [Helm 3](https://helm.sh/docs/intro/install/)
- Access to a Kubernetes cluster configured with `kubectl` (only needed to install or apply resources)
- For the optional EFS examples: an AWS EFS CSI driver and an EFS file system available to the cluster
- For the optional HTTPRoute example: the Gateway API CRDs and a compatible controller

## Helm chart

The chart is in [`app1/`](app1). By default it deploys three NGINX replicas behind a `ClusterIP` service on port 80. Ingress, Gateway API `HTTPRoute`, and horizontal pod autoscaling are disabled by default.

### Install

From the repository root, validate and preview the chart:

```shell
helm lint ./app1
helm template demo ./app1
```

Install or upgrade a release in its own namespace:

```shell
helm upgrade --install demo ./app1 --namespace demo --create-namespace
```

With the default `ClusterIP` service, access the application locally using port forwarding:

```shell
kubectl --namespace demo port-forward service/demo-app1 8080:80
```

Then open `http://127.0.0.1:8080`. If you change the release name or chart naming overrides, get the actual service name with `kubectl get services --namespace demo`.

Run the chart's connection test after installation:

```shell
helm test demo --namespace demo
```

Uninstall the release when finished:

```shell
helm uninstall demo --namespace demo
```

### Configuration

Override chart values with a values file (`-f my-values.yaml`) or `--set`:

| Value | Default | Description |
| --- | --- | --- |
| `replicaCount` | `3` | Number of replicas when autoscaling is disabled |
| `image.repository` | `nginx` | Container image repository |
| `image.tag` | `""` | Image tag; an empty value uses `appVersion` from `Chart.yaml` |
| `image.pullPolicy` | `IfNotPresent` | Kubernetes image pull policy |
| `service.type` | `ClusterIP` | Kubernetes Service type |
| `service.port` | `80` | Service and container HTTP port |
| `ingress.enabled` | `false` | Create an Ingress; configure its class, hosts, paths, and optional TLS under `ingress` |
| `httpRoute.enabled` | `false` | Create a Gateway API HTTPRoute; configure its parent references, hostnames, and rules under `httpRoute` |
| `autoscaling.enabled` | `false` | Create a HorizontalPodAutoscaler; configure replica bounds and target utilization under `autoscaling` |
| `resources` | `{}` | Container resource requests and limits |
| `serviceAccount.create` | `true` | Create a service account for the workload |

For example, install with two replicas and an explicit NGINX tag:

```shell
helm upgrade --install demo ./app1 --namespace demo --create-namespace --set replicaCount=2 --set image.tag=1.27
```

See [`app1/values.yaml`](app1/values.yaml) for all available settings, including probes, pod/container security contexts, volumes, scheduling options, and ingress or HTTPRoute details. Enable only the exposure options that are configured and supported by your cluster.

## Standalone Kubernetes examples

These manifests are independent of the Helm chart. Review each file and adjust names, namespaces, and environment-specific values before applying it.

| File | Contents |
| --- | --- |
| `01-ns-pods-svc.yaml` | Namespaces, NGINX pods, and Services |
| `pod-run-as-non-root.yaml` | Pod configured to run its container as a non-root user |
| `rback.yaml` | Namespace-scoped Role, RoleBinding, and ServiceAccount example |
| `testapp.yaml` | PVC, Deployment, and Service using an EFS-backed StorageClass |
| `efs-storage-class.yaml` | AWS EFS CSI StorageClass example; replace its file system ID with your own |
| `config-template.yaml` | Kubeconfig structure with placeholders for cluster and user values |

Apply a standalone manifest with:

```shell
kubectl apply -f <manifest-file>.yaml
```

For example, the EFS-backed application requires the EFS CSI driver, a correctly configured StorageClass, and a suitable EFS file system before its PVC can bind.

## Credentials and cluster safety

Treat kubeconfig files, tokens, certificates, and other cluster credentials as secrets. Do not publish or commit real credentials; use placeholders in examples and store credentials outside source control. Before applying a manifest, check its namespace and the resources it will create or change.
