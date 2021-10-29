# Kind Cheat Sheet

# Autocompletion

## Bash
```bash
# Setup the current shell
source <(kind completion bash)

########
## OR ##
########

# Update permanently
echo "source <(kind completion bash) >> ~/.bashrc"
```

## ZSH
```bash
source <(kind completion zsh)
```

# Basic
Create kind cluster named cncf-cheat-sheet
```bash
kind create cluster --name cncf-cheat-sheet
```

Create cluster and wait for all the components to be ready
```bash
kind create cluster --wait 2m
```

Get running clusters
```bash
kind get clusters
```

Delete kind cluster named cncf-cheat-sheet
```bash
kind delete cluster --name cncf-cheat-sheet
```

# Advanced Configuration
Use `kind.yaml` config file for more advanced use cases

## Ports
[Info](https://kind.sigs.k8s.io/docs/user/configuration/#extra-port-mappings)
Map port 80 from the cluster control plane to the host.

```bash
cat <<EOF | kind create cluster --name cncf-cheat-sheet --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
EOF
```


## Mount Directories
[Info](https://kind.sigs.k8s.io/docs/user/configuration/#extra-mounts)
Mount current directory into clusters control plane located at `/app`

* NOTE: MacOS users: make sure to share resources in docker-for-mac preferences 

```bash
cat <<EOF | kind create cluster --name cncf-cheat-sheet --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraMounts:
  - hostPath: .
    containerPath: /app
EOF
```

## Add Local Registry
[Info](https://kind.sigs.k8s.io/docs/user/local-registry/)
### Step 1

Create local registry
```bash
docker run -d --restart=always -p 127.0.0.1:5000:5000 --name cncf-cheat-sheet-registry registry:2
```

### Step 2

Create cluster
```bash
cat <<EOF | kind create cluster --name cncf-cheat-sheet --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:5000"]
    endpoint = ["http://cncf-cheat-sheet-registry:5000"]
nodes:
- role: control-plane
EOF
```
### Step 3

Connect registry with created network 
```bash
docker network connect kind cncf-cheat-sheet-registry
```

### Step 4

Update cluster about new registy
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: local-registry-hosting
  namespace: kube-public
data:
  localRegistryHosting.v1: |
    host: "localhost:5000"
EOF
```

## Multiple Workers
[Info](https://kind.sigs.k8s.io/docs/user/configuration/#nodes)
The default configuration will create cluster with one node (control-plane).

```bash
cat <<EOF | kind create cluster --name cncf-cheat-sheet --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
EOF
```
