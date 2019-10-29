# Scalable ML with Kubernetes and Kubeflow


# Setup

During this workshop we will use Minikube to host a local kubernetes environment, however any kubernetes deployment will do including other local options such as microk8s or cloud options such as GKE (Google), AKS (Azure), or EKS (AWS).



## kubectl

kubectl is a command line interface (CLI) for working with kubernetes clusters. It allows us to manage local **or** remote clusters after we're propperly registered and authenticated with the cluster.



#### Linux

```
gunzip ./utils/linux/kubectl.gz
chmod +x ./utils/linux/kubectl
sudo mv ./utils/linux/kubectl /usr/local/bin/kubectl
kubectl version
```

#### macOS


```
gunzip ./utils/mac/kubectl.gz
chmod +x ./utils/mac/kubectl
sudo mv ./utils/mac/kubectl /usr/local/bin/kubectl
kubectl version
```

## kfctl

kubectl:kubernetes :: kfctl:kubeflow

kfctl allows us to manage the deployment of kubeflow to our kubernetes cluster



#### Linux

```
cd ./utils/linux/
tar -xvf kfctl_v0.6.2_linux.tar.gz
sudo mv kfctl /user/local/bin/kfctl
```


#### macOS

```
cd ./utils/mac/
tar -xvf kfctl_v0.6.2_darwin.tar.gz
sudo mv kfctl /user/local/bin/kfctl
```

alternatively, [download from the releases page](https://github.com/kubeflow/kubeflow/releases/tag/v0.6.2) and add to your path as you see fit


## Minikube Setup


### Install Linux (Ubuntu)

1. Verify the output of this command is non-empty to check if your CPU supports virtualization:

```
grep -E --color 'vmx|svm' /proc/cpuinfo
```

2. Ensure you have a hypervisor installed. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) is recomended

3. Install Minikube

local file:

```
gunzip ./utils/linux/minikube.gz
chmod +x ./utils/linux/minikube
sudo install ./utils/linux/minikube /usr/local/bin
```

web:

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube

sudo install minikube /usr/local/bin/

```


### Install macOS


# Start Kubernetes Cluster

## Launch Minikube

```
$ minikube start --cpus 4 --memory 8096 --disk-size=40g --kubernetes-version v1.14.0 --insecure-registry "10.0.0.0/24"
```


### Local Registry Setup

```
$ minikube addons enable registry
```

#### 1A: Linux
```
$ sudo nano /etc/docker/daemon.json
```

add the following and save:

```json
{
  "insecure-registries" : ["localhost:5000"]
}
```

#### 1B: macOS 

* Navigate to your docker preferences 
* Under 'Daemon' add localhost:5000 to the list of insecure registries


#### 2: Port-forward to minikube registry

If you're using the minikube registry addon, we'll need to create a tunnel between our localhost:5000 and the registry hosted on our minikube cluster. We can do this quickly by running a lightweight alpine linux docker image:


In a _new_ terminal, run:
```
# load local file for the alpine image w/ socat installed
docker import ./containers/alpine.tar alpine

# run the container 
sudo docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TC$(minikube ip):5000"

```

## Kubeflow Setup

We have to create the _kubeflow-anonymous_ namespace manually due to a small issue with the bootstrap process (should be fixed in future releases)

```
kubectl create namespace kubeflow-anonymous
```

Use existing configuration + cached files
``` 
cd kfdemo
kfctl apply all -V
```

Download and build from web
``` 
export KFAPP='kfdemo'
export CONFIG='https://raw.githubusercontent.com/kubeflow/kubeflow/v0.6-branch/bootstrap/config/kfctl_k8s_istio.0.6.2.yaml'

kfctl init ${KFAPP} --config=${CONFIG} -V

kubectl create namespace kubeflow-anonymous

cd ${KFAPP}
kfctl generate all -V
kfctl apply all -V

```

## Access Kubeflow UI

```
export NAMESPACE=istio-system
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```


## Seldon Setup

### Helm + Tiller

#### Linux
```
tar -zxvf ./utils/linux/helm-v2.15.1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

#verify
helm help
```

#### macOS

Local File:
```
tar -zxvf ./utils/mac/helm-v2.15.1-darwin-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm


#verify
helm help
```


Homebrew
```
brew install kubernetes-helm

#verify
helm help
```

#### Initialize Helm on your Kubernetes Cluster

```
helm init --history-max 200
```

### Amassador + SeldonCore

Use Helm to Install Ambassador for ingress management:

```
helm install stable/ambassador --name ambassador --set crds.keep=false
```

If you installed Kubeflow first:
```
kubectl delete customresourcedefinition seldondeployments.machinelearning.seldon.io
```

Use Helm to install SeldonCore
```
helm install seldon-core-operator --name seldon-core --repo https://storage.googleapis.com/seldon-charts --set usageMetrics.enabled=true --namespace seldon-system --set ambassador.enabled=true
```



## Other Commands

```
minikube mount $(pwd)/pv_storage/:/mkdata

kubectl port-forward $(kubectl get pods -l app.kubernetes.io/name=ambassador -o jsonpath='{.items[0].metadata.name}') 8003:8080


```