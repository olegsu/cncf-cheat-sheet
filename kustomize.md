# Kustomize Cheat Sheet

kustomize is a command line tool supporting template-free, structured customization of declarative configuration targeted to k8s-style objects.

Targeted to k8s means that kustomize has some understanding of API resources, k8s concepts like names, labels, namespaces, etc. and the semantics of resource patching.

kustomize is an implementation of DAM.
[source](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#kustomize)

* [Github](https://github.com/kubernetes-sigs/kustomize)
* [Docs](https://kustomize.io/)

# Installation
```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

# Basic Usage

## Built Into kubectl
Kustomize is build into `kubectl`.
`kubectl apply -k` acts the same was as `kustomize build path/to/some/app | kubectl apply -f -`.

## Create Base

### From Kubernetes Manifests
Search all kubernetes resources in the current directory and add them to `resources`
```bash
kustomize create --autodetect
```

### From Kustomization Base
Use `path/to/base` of another kustomization overlay as base to the new one
```bash
kustomize create --resources path/to/base
```

Search current directory and all sub-directories
```bash
kustomize create --resources path/to/base --recursive
```

## Build Resources

### URL
```bash
kustomize build https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld?ref=v1.0.6
```

## Modifiying Base

### Labels & Annotations

Create new kustomization.yaml from remote resources, adding to all the resources labels `app=hello-world` and `cloud=gcp`
```bash
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6 --labels app:hello-world,cloud:gcp
kustomize build .
```

### Namespace

Create new kustomization.yaml from remote resources, adding `namespace` attribute to all its resources
```bash
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6 --namespace dev
kustomize build .
```

### Resource Names

Update name prefix for all the resources
```bash
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit set nameprefix temp
kustomize build .
```

### Images

Update the image `monopole/hello:1` to `monopole/hello:latest`
```bash
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit set image monopole/hello:1=monopole/hello:latest
kustomize build .
```

### ConfigMap & Secret

Generate ConfigMap manifests from variables
```bash
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit add configmap hello-world --behavior create --from-literal=host=google.com
kustomize build .
```

### JSON Patch

Update the number of the replicas
```bash
cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 1
EOF
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit add patch --kind Deployment --path patch.yaml
kustomize build .
```

### Strategic Merge

Add `emptyDir` volume 
```bash
cat <<EOF > patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  template:
    spec:
      containers:
      - name: the-container
        volumeMounts:
        - name: emptyDir
          mountPath: /appdata 
    volumes:
    - name: emptyDir
      emptyDir: {}
EOF
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit add patch --kind Deployment --path patch.yaml
kustomize build .
```

# Remove Resource

Completly omit resources from the final manifest set
```bash
cat <<EOF > patch.yaml
\$patch: delete
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
EOF
kustomize create --resources https://github.com/kubernetes-sigs/kustomize.git/examples/helloWorld\?ref\=v1.0.6
kustomize edit add patch --kind ConfigMap --path patch.yaml
kustomize build .
```