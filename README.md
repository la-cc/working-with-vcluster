# Overview

*_NOTE (28.03.2023)_*: In the exchange with a contributor of loft-sh it turned out that the requested solution should run in rootless mode by default. In addition, the documentation will soon be adapted to the missing values.  

This is a short tutorial on how to get vcluster deployed with the K3s distribution in rootless mode (running as non-root user).

There are some errors in the official documentation.
The documentation is complete, but spread over different steps.
Furthermore the necessary values like `fsGroup: 12345` and `securityContext.runAsGroup: 12345` are not declared in the values.yaml of the chart.
You can find these out if you take a closer look at the templates.


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

```
git clone git@github.com:la-cc/working-with-vcluster.git
```

## 2. Overwrite the values in values.yaml

_NOTE (28.03.2023):_ TLS termination works even without a valid certificate. The vcluster then terminates it via the fake certificate, which has been signed by the kubernetes cluster anyway. This means that authentication is still possible.

If you don't want use a ingress to the vcluster, then set `ingress.enabled: false` and delete the `syncer` config.

If you want use a ingress with SSL passthrough as TLS termination with a valid cert to connect to the vcluster, then please make sure you have enabled the SSL passthrough feature, which is disabled by default. For the nginx ingress controller you have only to set `--enable-ssl-passthrough` to enables the [SSL Passthrough](https://kubernetes.github.io/ingress-nginx/user-guide/tls/?_gl=1*1ircp4t*_ga*MTQzOTI4NzczOS4xNjc5ODMxNDIw*_ga_4RQQZ3WGE9*MTY3OTkwNzA5Ni41LjEuMTY3OTkwNzE5MS4yNy4wLjA.#ssl-passthrough) feature, which is disabled by default.



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
