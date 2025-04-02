<p align="center">
   <h1 align="center">LW-Tasks</h1>
</p>

## 📝 1. SSH into VM and do the task mentioned below

This is your VM address = 64.227.139.218

This is your username: root

1. Please SSH into the VM and install
   - A local Kubernetes cluster using https://k3s.io/
   - neovim
   - zsh
   - git
   - kubectl
   - terraform
   - helm

2. Then do the following

- using git transfer the following repo to your VM https://github.com/bitnami/charts.git
- using neovim edit values.yaml of nginx chart in bitnami/ directory
- update / save replicaCount to 2
- helm install into your local k8s cluster
- kubectl to check k8s output of 2 replicas running

### Step 1: SSH into VM
Connect to the VM using this command in your terminal

```
ssh -i /Users/imran/LWSSHkey root@64.227.139.218
```
### Step 2: Install required tools/packages
#### Install k3s
```
curl -sfL https://get.k3s.io | sh -
```
Check for ready node and verify installation

```
sudo k3s kubectl get node
```

#### Install neovim

```
sudo apt install neovim
```

#### Install zsh

```
sudo apt install zsh
```

#### Install git

```
sudo apt install git
```

#### Install Kubectl

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

#### Install Terraform

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

#### Install Helm

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

#### Configure helm to connect with k3s cluster

- Copy the K3s cluster configuration file to your local machine
```
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

- Set Kubernetes Context
```
kubectl config use-context default
```
### Step 3: Clone the Bitnami charts repo

```
git clone https://github.com/bitnami/charts.git
```

### Step 4: Change directory and go to charts/bitnami/nginx/

```
cd charts/bitnami/nginx/
```

### Step 5: Open values.yaml using neovim and edit the replicaCount value to 2

```
nvim values.yaml
```
Search for replicaCount and change the value from 1 to 2 And save the file

### Step 6: Fetch missing dependencies
```
helm dependency build
```

### Step 7: Install and deploy nginx cluster using helm
```
helm install my-nginx ./
```

---

## 📝 2. choose an opensource observability stack and deploy it

### :x: Getting error while connecting to the VM

After completing the first task, David shared the credentials for another VM and asked to perform the first task again there. But I was not able to SSH into it. And then Walter and Anthony checked the VM to troubleshoot the issue. Then me, Walter and Anthony tried to find the issue on call.

I checked the following log files `var/log/auth.log` , `var/log/syslog` with the help of `grep` and `tail` commands for the reason of error while connecting to the VM. I could see that ssh serves went down because of multiple failed attempts of authentication. Thats why I was not able to access the VM. And when you restarted the VM the server and port 22 opened again and thats when I was able to connect again.

And i took whole day. In the end we found out that the policies applied by Walter took some time to be efective and then I was able to access the VM.

this is my finding. please let me know if there is anything else as well.

### Step 1: Install Prometheus using Helm

- Add and update Prometheus Helm repository and get the Prometheus Helm chart.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

- Install Prometheus

```
helm install prometheus prometheus-community/prometheus
```

### Step 2: Exposing the Prometheus-server service on Kubernetes
- To access the Promethues server application on our browser we have to expose the service.

- To expose the service run this command

```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

### Step 3: Install Grafana using Helm

- Add and update Grafana Helm repository and get the Grafana Helm chart.

```
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
```

- Install Grafana

```
helm install grafana grafana/grafana
```

### Step 4: Exposing the Grafana service on Kubernetes

- To access the Grafana server application on our browser we have to expose the service.

- To expose the service run this command

```
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```

### Step 5: Access Grafana on browser and login
- Default username will be `admin` and to get the password run the following command

```
kubectl get secret --namespace monitoring my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

- Login to Grafana and add Prometheus as data source

- Then go to Dashboards and import a Kubernetes prometheus dashboard to visualize all the metrics from Prometheus into a Grafana dashboard. 

---

## 📝 3. Introduce Grafana Tempo and OpenTelemetry and use it with a test application
In this task we will implement grafana tempo with local as object storage and open telemetry with auto-instrumentation using open telemetry operator for Google Cloud's Online Boutique microservices demo application running on Kubernetes. This will provide you with distributed tracing capabilities across the entire application.

### Prerequisites

- Kubernetes cluster up and running
- `kubectl` configured to access your cluster
- Helm installed
- Git to clone the Online Boutique repository

### Step 1: Clone the Online Boutique Repository

```
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
cd microservices-demo
```

### Step 2: Install Grafana Tempo with Local Storage

- First, let's create a namespace for our observability stack:

```
kubectl create namespace observability
```

- Create a configuration values file for Tempo named tempo-values.yaml

```
tempo:
  storage:
    trace:
      backend: local
      local:
        path: /var/tempo/traces
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 1
      memory: 1Gi
persistence:
  enabled: true
  size: 10Gi
```

- Now use Helm with the values file to install Grafana Tempo

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install tempo grafana/tempo -n observability -f tempo-values.yaml
```

### Step 3: Install OpenTelemetry Operator




### :bulb: Interesting and new things I learned until these step.
I got to learn about:
- Traces
- Span
- Difference between **Traces** , **Metrics** and **Logs**
- These 3 are known as 3 pillars of observability

