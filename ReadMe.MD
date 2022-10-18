### 1. Creating a K8S Cluster

Run the following command to create a K8S cluster named 'next-try-out-cluster' on AWS.

```eksctl create cluster --name next-try-out-cluster --version 1.23 --region eu-west-1 --nodegroup-name worker-nodes --node-type t2.large --nodes 2 --nodes-min 2 --nodes-max 3```

### 2. Prerequisites


#### 2.1. Add the onf helm repo
```helm repo add onf https://charts.opencord.org```
#### 2.2. Update the helm repo 
```helm repo update```


### 3. Install VOLTHA infrastructure

#### 3.1. via helm-chart
```helm upgrade --install --create-namespace -n infra voltha-infra onf/voltha-infra```

#### 3.2. Accessing ONOS GUI
```kubectl port-forward -n infra svc/voltha-infra-onos-classic-hs 8181```

Then open a browser and access the dashboard with the address below:

```http://localhost:8181/onos/ui/index.html```

```default username : onos```

```default password: rocks```


### 4. Deploy VOLTHA

#### 4.1. via helm-chart
```helm upgrade --install --create-namespace -n voltha voltha onf/voltha-stack --set global.stack_name=voltha --set global.voltha_infra_name=voltha-infra --set  global.voltha_infra_namespace=infra```


### 5. Deploy BBSim

#### 5.1. via helm-chart
```helm upgrade --install -n voltha bbsim0 onf/bbsim --set olt_id=10```

### 6. Installing and Configuring voltctl

```
HOSTOS="$(uname -s | tr "[:upper:]" "[:lower:"])"
HOSTARCH="$(uname -m | tr "[:upper:]" "[:lower:"])"
if [ "$HOSTARCH" == "x86_64" ]; then
    HOSTARCH="amd64"
fi
sudo wget https://github.com/opencord/voltctl/releases/download/v1.7.4/voltctl-1.7.4-$HOSTOS-$HOSTARCH -O /usr/local/bin/voltctl
sudo chmod +x /usr/local/bin/voltctl
source <(voltctl completion bash)
```

In order to use voltctl you need to expose the voltha-api service:

```kubectl -n voltha port-forward --address 0.0.0.0 svc/voltha-voltha-api 55555```


### Deleting the K8S Cluster

```eksctl delete cluster --region=eu-west-1 --name=next-try-out-cluster```


### Resources
1. https://github.com/opencord/voltha-helm-charts 
2. [How to setup Prometheus, Grafana dashboard for Kubernetes monitoring on AWS - Rahul Wagh](https://jhooq.com/prometheous-k8s-aws-setup/)


