# Overview

This is a short tutorial on how to get vcluster deployed with the K3s distribution in rootless mode (running as non-root user).

There are some errors in the official documentation.
The documentation is complete, but spread over different steps.
Furthermore the necessary values like `fsGroup: 12345` and `securityContext.runAsGroup: 12345` are not declared in the values.yaml of the chart.
You can find these out if you take a closer look at the templates.

The further goal was also to enable access to the cluster via an ingress that is terminated via SSL passthrough.

# vcluster k3s

## 0. Preparation:

### Install vcluster cli

Mac (Intel/ADM):

```
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-darwin-amd64" && sudo install -c -m 0755 vcluster /usr/local/bin && rm -f vcluster
```

Mac (Silicon/ARM):

```
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-darwin-arm64" && sudo install -c -m 0755 vcluster /usr/local/bin && rm -f vcluster
```

Linux (AMD):

```
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64" && sudo install -c -m 0755 vcluster /usr/local/bin && rm -f vcluster
```

Linux (ARM):

```
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-arm64" && sudo install -c -m 0755 vcluster /usr/local/bin && rm -f vcluster
```

## 1. Clone or Fork this Repository

## 2. Overwrite the values in values.yaml

If you don't want use a ingress with SSL passthrough as TLS termination to connect to the vcluster, then set `ingress.enabled: false` and delete the `syncer` config.

```
ingress:
  enabled: true
  host: vcluster-k3s.<YOUR-DOMAIN>
  clusterIssuer: <YOUR-CLUSTER-ISSUER>


  # Syncer configuration
  syncer:
    extraArgs:
      - --tls-san=vcluster-k3s.<YOUR-DOMAIN>
```

## 3. Helm: install dependencies

```
helm dependency build working-with-vcluster/helm/k3s
```

## 4. Helm: template the helm chart

```
helm template --namespace vcluster-k3s --name-template=vcluster-k3s  working-with-vcluster/helm/k3s
```

## 5. vcluster: connect to the cluster

If you disable `ingress.enabled: false`, then skip the next steps and check the doc [In-Cluster](https://www.vcluster.com/docs/operator/external-access#in-cluster) how to connect against the vcluster.

```
vcluster connect vcluster-k3s -n vcluster-k3s --server=https://vcluster-k3s.<YOUR-DOMAIN>
```

## 6. Check namespace

```
kubectl get ns

#You will get an output like:
NAME              STATUS   AGE
default           Active   57m
kube-system       Active   57m
kube-public       Active   57m
kube-node-lease   Active   57m
```

# References:

- [vcluster k3s helm chart](https://github.com/loft-sh/vcluster/tree/main/charts/k3s)
- [Quickstart Guide](https://www.vcluster.com/docs/quickstart)
- [Exposing vcluster (ingress etc.)](https://www.vcluster.com/docs/operator/external-access)
