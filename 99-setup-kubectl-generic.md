# Setup kubectl

It is assumed that you are provided with a kubernetes cluster by the instructor. Before you are able to do anything on the cluster, you need to be able to *talk* to this cluster from/using your computer. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - short for Kubernetes Controller - is *the* command line tool to talk to a Kubernetes cluster. To get that on your computer, you will need to follow instruction from [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/). We have provided you simplified steps:

## MAC OS:
```
brew install kubectl
```

OR

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```

## Linux (RedHat/CentOS/Fedora):

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```

OR

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

## Debian/Ubuntu:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
```

OR

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```



Kubectl is a *go* binary which allows you to execute commands on your cluster. Where, "your cluster" could be: a single node VM, such as [minikube](https://github.com/kubernetes/minikube), or a set of VMs on your local computer or somewhere on a host in your data center, a bare-metal cluster, or a cluster provided by any of the cloud providers - as a service - such as GCP.

**Note:** Due to restrictions with virtualization inside a virtual machine (nested virtualization), you cannot run minikube on cloud VMs. Minikube is a part of the Kubernetes open source project, with the single goal of getting a simple cluster up and running with just one virtual machine acting as both master-node and worker-node.

For the remainder of this workshop, we assume you have a Kubernetes cluster on google cloud.


# Authenticate to your Google k8s cluster:
In some workshops we provide a preconfigured **config** file, which you simply save as ~/.kube/config on your local computer, and you are golden.
However, in other setups, you may need to actually authenticate to the cluster, and get a config file directly from GCP. Actually, the instructors may also need to use/do the following steps when they are setting up individual VMs for each student. Either way, the access to cluster can only be obtained after authenticating to google. To authenticate against your cluster, you will need google cloud SDK installed on your computer, and a gmail account. Assuming you already have a gmail account, we proceed to install google SDK on the local computer. To do that we have provided a summary of steps you need to perform. For more details, visit [this URL](https://cloud.google.com/sdk/install) .


## MAC OS:

```
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init
```

## Linux (Debian/RedHat):
```
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init
```

OR

## Linux (Debian/Unbuntu):
```
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install google-cloud-sdk
gcloud init
```


## Linux (RedHat/CentOS/Fedora):
```
sudo tee -a /etc/yum.repos.d/google-cloud-sdk.repo << EOM
[google-cloud-sdk]
name=Google Cloud SDK
baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOM

yum install google-cloud-sdk
gcloud init
```


## Windows:
Follow these instructions: [https://cloud.google.com/sdk/docs/downloads-interactive#windows](https://cloud.google.com/sdk/docs/downloads-interactive#windows)


Then, run:
```
 # cluster connection via service account
 # Create key file on your vm - the instructor will mail you the contents
vi keyfile.json

 # authenticate with cloud
gcloud auth activate-service-account --key-file keyfile.json

 # Get the cluster credentials for kubectl
gcloud container clusters get-credentials cluster-london --zone europe-west1-b --project praqma-education
```

Google will do some magic under the hood, which does a few things:
* Fetches certificates and tokens (secrets)
* Puts them into the Kubernetes configuration file, located at ~/.kube/config

## Verify configuration:
You can verify this by looking at the config file using the following command:

```
kubectl config view
```

You should see something like this:

```
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://1.2.3.4
  name: gke_praqma-education_europe-west1-b_dcn-cluster-35
contexts:
- context:
    cluster: gke_praqma-education_europe-west1-b_dcn-cluster-35
    user: gke_praqma-education_europe-west1-b_dcn-cluster-35
  name: gke_praqma-education_europe-west1-b_dcn-cluster-35
current-context: gke_praqma-education_europe-west1-b_dcn-cluster-35
kind: Config
preferences: {}
users:
- name: gke_praqma-education_europe-west1-b_dcn-cluster-35
  user:
    password: secret-password-ea4a2fb76dc9
    username: admin
```


You should now have access to the google cloud cluster! Verify by looking at the nodes for the cluster:

```
kubectl get nodes
```

You should be able to see something similar to what is shown below:
```
$ kubectl get nodes
NAME                                             STATUS    ROLES     AGE       VERSION
ip-172-20-40-108.eu-central-1.compute.internal   Ready     master    1d      v1.8.0
ip-172-20-49-54.eu-central-1.compute.internal    Ready     node      1d      v1.8.0
ip-172-20-60-255.eu-central-1.compute.internal   Ready     node      1d      v1.8.0
```

If you add the `-o wide` parameters to the above command, you will also see the public IP addresses of the nodes:

```
$ kubectl get nodes -o wide
NAME                                            STATUS    ROLES     AGE       VERSION        EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-dcn-cluster-35-default-pool-dacbcf6d-3918   Ready     <none>    17h       v1.8.8-gke.0   35.205.22.139   Container-Optimized OS from Google   4.4.111+         docker://17.3.2
gke-dcn-cluster-35-default-pool-dacbcf6d-c87z   Ready     <none>    17h       v1.8.8-gke.0   35.187.90.36    Container-Optimized OS from Google   4.4.111+         docker://17.3.2
```

**Note:** On Kubernetes clusters provided by a Kubernetes service provider, you will only see worker nodes as a result of executing the above command. On other clusters, you will see both master and worker nodes.

```
$ kubectl get nodes -o wide
NAME                                             STATUS    ROLES     AGE     VERSION   EXTERNAL-IP     OS-IMAGE                      KERNEL-VERSION   CONTAINER-RUNTIME
ip-172-20-40-108.eu-central-1.compute.internal   Ready     master    1d      v1.8.0    1.2.3.4         Debian GNU/Linux 8 (jessie)   4.4.78-k8s       docker://1.12.6
ip-172-20-49-54.eu-central-1.compute.internal    Ready     node      1d      v1.8.0    2.3.4.5         Debian GNU/Linux 8 (jessie)   4.4.78-k8s       docker://1.12.6
ip-172-20-60-255.eu-central-1.compute.internal   Ready     node      1d      v1.8.0    5.6.7.8         Debian GNU/Linux 8 (jessie)   4.4.78-k8s       docker://1.12.6
```



**Note:** Depending on the setup for this workshop, you may not be the only tenant on the cluster; you may be sharing it with the rest of the people around you in the course! So be careful!
