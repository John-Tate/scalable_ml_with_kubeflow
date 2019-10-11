# Scalable Machine Learning with Kubernetes and Kubeflow

## **K8s Setup**

### Install Kubectl

Kubectl is a cli tool that allows you to interact with any kubernetes cluster, whether in the cloud, on-prem, or on your local machine


### Start a cluster

Cloud
- [GCP](google.com)
- [Azure](/Azure.md)
- [AWS](aws.com)

Local
- Microk8s
- MiniKube
- MiniKF




## **Install Kubectl**

- macOS

```
brew install kubectl 
```

- Linux

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl
```


## **Kubeflow Setup**

### Install kfctl

Download:
- MacOS
```
wget https://github.com/kubeflow/kubeflow/releases/download/v0.6.2/kfctl_v0.6.2_darwin.tar.gz
```

- Linux
```
wget https://github.com/kubeflow/kubeflow/releases/download/v0.6.2/kfctl_v0.6.2_linux.tar.gz
```

Unpack
```
tar -xvf kfctl_<release tag>_<platform>.tar.gz
```

### Initialize KF Manifests

```
export KFAPP=<your choice of application directory name> (ensure this is lowercase)

export CONFIG=https://raw.githubusercontent.com/kubeflow/kubeflow/v0.6.2/bootstrap/config/kfctl_k8s_istio.0.6.2.yaml

kfctl init ${KFAPP} --config=${CONFIG}
```
Generate and Deploy:
```
cd ${KFAPP}
kfctl generate all -V
kfctl apply all -V
```