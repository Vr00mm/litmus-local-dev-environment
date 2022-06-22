# litmus-local-dev-environment

<p>Hello, here are few instruction to deploy a local kubernetes / litmus env for dev !!!</p>

### Install curl, jq and git

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
### Finally restart your computer

```
reboot
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

<p> Update kubeconfig context to target minikube</p>

```
cp ~/.kube/config{,_backup}
minikube update-context
```

<p> Lets try to reach our cluster</p>

```
kubectl get node
```

<p>We're now ready to deploy litmus !!!</p>

## Deploy litmus

<p> Download helm chart </p>

```
cd ~
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
echo http://${MINIKUBE_IP}:${LITMUS_PORT}

```
<p>Use control + click on link to open in your browser</p>


<p>Wait for the agent active</p>

```
kubectl -n litmus wait pods --for condition=Ready -l name
kubectl -n litmus wait pods --for condition=Ready -l app
```

<p>we're ready to dev :)</p>

## Example for subscriber

<p>Download litmus</p>

```
cd ~
git clone https://github.com/litmuschaos/litmus
cd litmus/litmus-portal/cluster-agents/subscriber
```

<p>Build docker image</p>

```
docker build . -t subscriber:dev
```

<p>Scale off the current subscriber</p>

```
kubectl -n litmus scale deployment subscriber --replicas=0
```

<p>create env-file, you need update this env-file after each reboot</p>

```
echo -n > env-file
echo "AGENT_NAMESPACE=litmus" >> env-file
echo ACCESS_KEY=$(kubectl -n litmus get secret/agent-secret -ojson | jq -r '.data.ACCESS_KEY' |base64 -d) >> env-file
echo CLUSTER_ID=$(kubectl -n litmus get secret/agent-secret -ojson | jq -r '.data.CLUSTER_ID' |base64 -d) >> env-file
echo AGENT_SCOPE=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.AGENT_SCOPE') >> env-file
echo COMPONENTS=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.COMPONENTS') >> env-file
echo CUSTOM_TLS_CERT=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.CUSTOM_TLS_CERT') >> env-file
echo IS_CLUSTER_CONFIRMED=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.IS_CLUSTER_CONFIRMED') >> env-file
echo SERVER_ADDR=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.SERVER_ADDR') >> env-file
echo SKIP_SSL_VERIFY=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.SKIP_SSL_VERIFY') >> env-file
echo START_TIME=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.START_TIME') >> env-file
echo VERSION=$(kubectl -n litmus get cm/agent-config -ojson | jq -r '.data.VERSION') >> env-file
```
