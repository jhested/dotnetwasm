# Running .net wasm in kubernetes

This project aims to demonstrate how to deploy .net wasm containers to Kubernetes.

## Prerequisites

- [Docker Desktop or Docker Engine](https://docs.docker.com/desktop/)
- [MicroK8S](https://microk8s.io/#install-microk8s) with [kwasm](https://microk8s.io/docs/addon-kwasm)
- [wasmtime](https://github.com/bytecodealliance/wasmtime)

## Create .net wasm app

See https://github.com/dotnet/dotnet-wasi-sdk for more details

To get started you can create a simple .net console app with

```bash
dotnet new console -o WasmConsoleApp
cd WasmConsoleApp
dotnet add package Wasi.Sdk --prerelease
dotnet build
```

This should produce a wasm binary under `bin/Debug/net7.0/WasmConsoleApp.wasm`. 

You can try it out by running `wasmtime bin/Debug/net7.0/WasmConsoleApp.wasm`

```
$ wasmtime bin/Debug/net7.0/WasmConsoleApp.wasm 
Hello, World!
```

## Create and push wasm container image

Add a Dockerfile with content, more info on scratch [here ](https://hub.docker.com/_/scratch)

```
FROM --platform=$BUILDPLATFORM scratch
COPY ./bin/Debug/net7.0/WasmConsoleApp.wasm /WasmConsoleApp.wasm
ENTRYPOINT [ "WasmConsoleApp.wasm" ]
```

Push image to docker hub

```
docker login
docker buildx build -t dotnet-wasm:latest .
docker image tag dotnet-wasm:latest <username>/dotnet-wasm:latest
```

## Deploy to kubernetes

Create a new deployment file called `deployment.yaml`

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: crun
handler: crun
---
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: wasm-test
spec:
  template:
    metadata:
      annotations:
        module.wasm.image/variant: compat-smart
      creationTimestamp: null
    spec:
      containers:
      - image: <username>/dotnet-wasm:latest
        name: wasm-test
        resources: {}
      restartPolicy: Never
      runtimeClassName: crun
  backoffLimit: 1
```

Apply the file to your local cluster

```
microk8s kubectl apply -f deployment.yaml
```

Verify log output from pod

## References 
- [MicroK8s](https://microk8s.io/)
- [Getting started with Docker + Wasm](https://nigelpoulton.com/getting-started-with-docker-and-wasm/)
- [WebAssembly on Kubernetes](https://nigelpoulton.com/webassembly-on-kubernetes-everything-you-need-to-know/)
- [wasmtime](https://github.com/bytecodealliance/wasmtime)
- [wasm-to-oci](https://github.com/engineerd/wasm-to-oci)
