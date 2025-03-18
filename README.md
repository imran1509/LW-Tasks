# LW-First-Task
64.227.139.218

username: root

- Please SSH into the VM and install
   - A local Kubernetes cluster using https://k3s.io/
   - neovim
   - zsh
   - git
   - kubectl
   - terraform
   - helm

Then do the following

- using git transfer the following repo to your VM https://github.com/bitnami/charts.git
- using neovim edit values.yaml of nginx chart in bitnami/ directory
- update / save replicaCount to 2
- helm install into your local k8s cluster
- ubectl to check k8s output of 2 replicas running

Make notes on your process. Please try not to use GPT/LLM for this. Googling is fine.
The journey is more important then the end result.

## Step 1: SSH into VM
Connect to the VM using this command in your terminal

```
ssh -i /Users/imran/LWSSHkey root@64.227.139.218
```
## Step 2: Install required tools/packages
### Install k3s
```
curl -sfL https://get.k3s.io | sh -
```
Check for ready node and verify installation

```
sudo k3s kubectl get node
```

### Install neovim

```
sudo apt install neovim
```

### Install zsh

```
sudo apt install zsh
```

### Install git

```
sudo apt install git
```

### Install Kubectl

- Download the latest release
```
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

- Install Kubectl
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- Check version and verify installation
```
kubectl version --client
```

### Install Terraform

- Install unzip
```
sudo apt install unzip
```
- Confirm latest version from the terraform website "https://www.terraform.io/downloads.html"
- Download the latest version of Terraform
```
wget https://releases.hashicorp.com/terraform/1.11.2/terraform_1.11.2_linux_amd64.zip
```

- Extract the downloaded file

```
unzip terraform_1.11.2_linux_amd64.zip
```

- Move the extracted file into /usr/local/bin
```
sudo mv terraform /usr/local/bin/
```

- Run Terraform

```
terraform --version 
```

### Install Helm

- fetch and download the Helm installer script
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

- Give executable permission to the script
```
chmod 700 get_helm.sh
```

- Run the script to install helm
```
./get_helm.sh
``` 
     
