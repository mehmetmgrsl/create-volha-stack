### 1. Creating a K8S Cluster

Run the following command to create a K8S cluster named 'next-try-out-cluster' on AWS.

```eksctl create cluster --name next-try-out-cluster --version 1.25 --region eu-west-1 --nodegroup-name worker-nodes --node-type t2.large --nodes 2 --nodes-min 2 --nodes-max 3```

### 2. Prerequisites


#### 2.1. Add the onf helm repo
```helm repo add onf https://charts.opencord.org```
#### 2.2. Update the helm repo 
```helm repo update```


### 3. Install VOLTHA infrastructure

#### 3.1. via helm-chart
##### 3.1.1. If you have a K8S cluster >= v1.25
###### Apply ->  https://github.com/opencord/voltha-helm-charts/issues/5#issuecomment-1466006762 
###### Run -> ```helm upgrade --install --create-namespace -n infra voltha-infra ./voltha-infra/```
##### 3.1.2. If you have a K8S cluster < v1.25
###### Run -> ```helm upgrade --install --create-namespace -n infra voltha-infra onf/voltha-infra```

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


### 7. Sanity Checks

As first make sure that all components are running correctly:

```shell
$ kubectl get pods -A
NAMESPACE     NAME                                                            READY   STATUS    RESTARTS      AGE
infra         bbsim-sadis-server-7547584ff6-nhf8f                             1/1     Running   0             17m
infra         voltha-infra-etcd-0                                             1/1     Running   0             17m
infra         voltha-infra-freeradius-859fb6c4c9-jqlbs                        1/1     Running   0             17m
infra         voltha-infra-kafka-0                                            1/1     Running   1 (16m ago)   17m
infra         voltha-infra-onos-classic-0                                     1/1     Running   0             17m
infra         voltha-infra-onos-classic-onos-config-loader-7b578bfcb4-cjr5k   1/1     Running   0             17m
infra         voltha-infra-zookeeper-0                                        1/1     Running   0             17m
voltha        bbsim0-57b6dd9d77-nbr6n                                         1/1     Running   0             14m
voltha        voltha-voltha-adapter-openolt-58b78b5fd-gkfz4                   1/1     Running   0             14m
voltha        voltha-voltha-adapter-openonu-69b8d644b-66vpx                   1/1     Running   0             14m
voltha        voltha-voltha-ofagent-7459fccc84-gfmg8                          1/1     Running   0             14m
voltha        voltha-voltha-rw-core-677bb9488d-ht8wd                          1/1     Running   0             14m
```

#### 7.1. Provisioning an OLT

Once you completed the `Sanity Checks` you can provision an OLT.

To create and enable the OLT device in VOLTHA you can use `voltctl` commands below:

```shell
voltctl device create -t openolt -H bbsim0.voltha.svc:50060
voltctl device list --filter Type~openolt -q | xargs voltctl device enable
```

Once the OLT device is enabled you will see that an emulated ONU is reported to VOLTHA:

```shell
$ voltctl device list
ID                                      TYPE                 ROOT     PARENTID                                SERIALNUMBER    ADDRESS                       ADMINSTATE    OPERSTATUS    CONNECTSTATUS    REASON
8d861c7c-084b-49df-88bb-dcf3fa5060fc    openolt              true     70f03197-e663-49ef-b46c-4094ed2d29c1    BBSIM_OLT_10    &{bbsim0.voltha.svc:50060}    ENABLED       ACTIVE        REACHABLE        
15dad8f9-6bfe-4242-8377-d0571cb170c4    brcm_openomci_onu    false    8d861c7c-084b-49df-88bb-dcf3fa5060fc    BBSM000a0001    <nil>                         ENABLED       ACTIVE        REACHABLE        omci-flows-pushed
```

#### 7.2. Running the sanity tests

Define the KUBECONFIG and VOLTCONFIG env variables. i.e below:

```export KUBECONFIG=~/.kube/config```

```export VOLTCONFIG=~/.volt/config```


```git clone https://github.com/opencord/voltha-system-tests```

```make -C voltha-system-tests sanity-single-kind```


### 8. Deleting the K8S Cluster

```eksctl delete cluster --region=eu-west-1 --name=next-try-out-cluster```


### Resources
1. https://github.com/opencord/voltha-helm-charts 
2. [How to setup Prometheus, Grafana dashboard for Kubernetes monitoring on AWS - Rahul Wagh](https://jhooq.com/prometheous-k8s-aws-setup/)


