# Installing KubeFlow
## Part 1: Installation
Following this [instructions](https://yann-leguilly.gitlab.io/post/2020-03-04-kubeflow-on-laptop/) (more less)

### 1. Multipass
If you will be runing this in windows you'll need to install Multipass. This step is optional for Windows.

```bash
# install multipass using the terminal
snap install multipass --classic
```

```bash
# Create your vm
multipass launch --name kubeflow-vm -c 3 -m 6G -d 125G
# Start your VM
multipass start kubeflow-vm
# Open all ports to the VM.
multipass exec kubeflow-vm -- sudo iptables -P FORWARD ACCEPT
# Log in the shell
multipass shell kubeflow-vm
```
Now you should be inside the shell of your VM.

> If you get the "Conection timeout issue" in Windows<br>
> Access CMD as admin and then run all of the following <br>
> **This will restart your PC.**

```bash
# Fix to the Conection timeout issue in Windows
ipconfig /release
ipconfig /flushdns
ipconfig /renew
netsh int ip set dns
netsh winsock reset
shutdown -r -f -t 10
```
### 2. microk8s v1.15
[Official instructions](https://microk8s.io/docs)

```bash
sudo snap install microk8s --classic --channel=1.15/stable
# Join the group
sudo usermod -a -G microk8s $USER
```

To make the changes available you need to do one of this:
- exit and comeback in to the shell , reset or
- log in to your user

```bash
# Let's log back in to the same user.
sudo chown -f -R $USER ~/.kube
su - $USER
```

```bash
# Check the status
microk8s status --wait-ready
```


### 3.1 Installing Kubeflow
#### 3.1.1 Preparing

```bash
microk8s.enable dns storage dashboard # run twice if it fails
microk8s.enable gpu # Enable GPU
```

#### 3.2 Connecting to the dashboard
```bash
token=$(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s.kubectl -n kube-system describe secret $token
microk8s.kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443 --address 0.0.0.0 &  
```
Go to:<br>
https://<ip>:10443<br>
https://localhost:10443
> Check the IP using _multipass list on the shell of your OS.

#### 3.3. Creating an alias to use kubectl
```bash
# Set the alias
sudo snap alias microk8s.kubectl kubectl
kubectl config view --raw > $HOME/.kube/config
```

#### 3.4. Installing kfctl
Set our current OS

```bash
export OPSYS=linux
```
Download and extract kfctl:
```bash
wget https://github.com/kubeflow/kfctl/releases/download/v1.0/kfctl_v1.0-0-g94c35cf_linux.tar.gz
tar -zvxf kfctl_v1.0-0-g94c35cf_linux.tar.gz
export PATH=$PATH:$PWD
```

#### 3.5. Deploying Kubeflow

Set the path to the base directory where you want to store one or more

```bash
# Select the name for your deployment
export KF_NAME=kf-deployment
# base dir for your deployment
export BASE_DIR=repos  
export KF_DIR=${BASE_DIR}/${KF_NAME}
# The configuration file to use when deploying Kubeflow.
export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.0-branch/kfdef/kfctl_k8s_istio.v1.0.0.yaml"  


# Create your Kubeflow configurations:
mkdir -p ~/${KF_DIR}
cd ~/${KF_DIR}
kfctl build -V -f ${CONFIG_URI}
export CONFIG_FILE=${KF_DIR}/kfctl_k8s_istio.1.0.0.yaml
kfctl apply -V -f  kfctl_k8s_istio.v1.0.0.yaml
```
#### 3.6. Monitoring the deployment
In order to check if everything is deployed properly, you can use the command:
```bash
kubectl -n kubeflow get po
```
It takes a while before everything is Running…

### 4. Connecting to Kubeflow
In order to connect to Kubeflow, you need the IP of you VM. First leave the shell in the VM by typing exit. Then:

```bash
multipass list
```

Then, in your browser go to that IP http://**[ ip of your vm ]**:31380

### 5. Setting Up kubectl Context
#### 5.1. Install kubectl

```bash
# On Linux
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

```bash
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```

### 5.2. Create the config file

```bash
multipass shell kubeflow-vm
```

Copy the config file from your VM
```bash
## Run this on your VM
cat ~/.kube/config # A
```
Copy the output of the above #A inside this file #B on your host
```bash
# Run this on your your host
mkdir ~/.kube/
nano ~/.kube/config # B
```

Check if it's working correctly
```bash
kubectl cluster-info
```

![img](https://yann-leguilly.gitlab.io/img/installing-kubeFlow/img12.png)
You can now control your kubernetes single node cluster from your regular environment (without connecting to the VM).
