# Linkerd

* [Github](https://github.com/linkerd/linkerd2)
* [Docs](https://linkerd.io/2.11/overview/)

## Installation

### Install linkerd
```bash
curl -fsL https://run.linkerd.io/install | sh
```

### Setup Cluster
```bash
kind create cluster --name cncf-cheat-sheet
linkerd check --pre
linkerd install | kubectl apply -f -
linkerd check
```

### Demo Application
```bash
curl -fsL https://run.linkerd.io/emojivoto.yml | kubectl apply -f -
```

# Autocompletion

## Bash
```bash
echo "source <(linkerd completion bash) >> ~/.bashrc"
```

## Zsh
```bash
source <(linkerd completion zsh)
```

# Basic

## Inject

Update the deployment annotations `linkerd.io/inject: enabled`. So the linkerd control plane knows to inject a side car during pod creation.
```bash
kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -
```

# Traffic Manipulation

## Split Traffic
Send 33% from the traffic that goes to `svc-a` into `svc-b`
```bash
cat <<EOF | kubectl apply -f -
apiVersion: split.smi-spec.io/v1alpha1
kind: TrafficSplit
metadata:
  name: traffic-split
spec:
  service: svc-a
  backends:
  - service: svc-a
    weight: 2
  - service: svc-b
    weight: 1
EOF
```

## Timeouts
Set HTTP timeout for 10 sec on all the requests that are coming into the service
```bash
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'all'
    conditions:
    - pathRegex: /*
    timeout: 30s
EOF
```


## Retries
For idempotent routes with an empty body:
```bash
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'route-name'
    conditions:
    - pathRegex: /*
    isRetryable: true
EOF
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: linkerd.io/v1alpha2
kind: ServiceProfile
metadata:
  name: svc-name.namespace.svc.cluster.local
  namepspace: default
spec:
  routes:
  - name: 'route-name'
    conditions:
    - pathRegex: /*
  retryBudget:
    retryRatio: 0.2
    minRetriesPerSecond: 10
    ttl: 10s
EOF
```

[More](https://linkerd.io/2.11/reference/service-profiles/) about service profiles


# Extensions

## Viz
Viz extension adds metrics stack into the cluster 
```bash
linkerd viz install | kubectl apply -f -
```

To start the web dashboard
```basd
linkerd viz dashboard
```




