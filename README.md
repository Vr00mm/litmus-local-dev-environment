# litmus-local-dev-environment

<p>Hello, here are few instruction to deploy a local kubernetes / litmus env for dev !!!</p>

### Install curl && jq && git
```
sudo apt update && sudo apt install -y curl jq git
```

### Install docker

<p> Install docker requirements </p>

```
sudo apt update && sudo apt install \
    ca-certificates \
    gnupg \
    lsb-release
```

<p> Add Docker’s official GPG key: </p>

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

<p> Set up the repository: </p>

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

<p> Finally install docker </p>

```
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io
```

<p> Add your user to docker group</p>

```
sudo usermod -aG docker $USER && newgrp docker
```

### Download && install minikube

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo install minikube /usr/local/bin/
```

### Download and install helm

```
cd /tmp
curl -fsSL -o helm.tgz https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz \
  && tar xzvf helm.tgz \
  && sudo install linux-amd64/helm /usr/local/bin/
```

## Run kubernetes local cluster

### Configure and run minikube cluster

<p> Lets configure our cluster ( customize it ) </p>

```
export CLUSTER_CPUS=4
export CLUSTER_RAM=8g
export CLUSTER_K8S_VERSION=v1.22.9
```

<p> And now, lets start our cluster </p>

```
minikube start --driver=docker \
  --cpus=${CLUSTER_CPUS} \
  --memory=${CLUSTER_RAM} \
  --kubernetes-version=${CLUSTER_K8S_VERSION}
```

<p> Check our cluster is started </p>

```
minikube status
```

## Interact with our minikube cluster

<p> Setup kubectl </p>

```
alias kubectl='minikube kubectl --'
```

<p> Lets try to reach our cluster</p>

```
kubectl get node
```

<p>We're now ready to deploy litmus !!!</p>

## Deploy litmus

<p> Download helm chart </p>

```
git clone https://github.com/litmuschaos/litmus-helm
cd litmus-helm
```

<p> Deploy litmus helm chart</p>

```
helm upgrade --install litmus charts/litmus --namespace litmus --create-namespace --wait
```

<p> Login to deploy self-agent</p>

```
MINIKUBE_IP=$(minikube ip)
LITMUS_PORT=$(minikube kubectl -- -n litmus get svc/litmus-frontend-service -ojson  | jq -r '.spec.ports[0].nodePort')
firefox http://${MINIKUBE_IP}:${LITMUS_PORT}
```
