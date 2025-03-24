# LW-First-Task
VM address = 64.227.139.218

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
- kubectl to check k8s output of 2 replicas running

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

### Configure helm to connect with k3s cluster

- Copy the K3s cluster configuration file to your local machine
```
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

- Set Kubernetes Context
```
kubectl config use-context default
```
## Step 3: Clone the Bitnami charts repo

```
git clone https://github.com/bitnami/charts.git
```

## Step 4: Change directory and go to charts/bitnami/nginx/

```
cd charts/bitnami/nginx/
```

## Step 5: Open values.yaml using neovim and edit the replicaCount value to 2

```
nvim values.yaml
```
Search for replicaCount and change the value from 1 to 2 And save the file

## Step 6: Fetch missing dependencies
```
helm dependency build
```

## Step 7: Install and deploy nginx cluster using helm
```
helm install my-nginx ./
```

## Error for not connecting to the VM
I checked the following log files `var/log/auth.log` , `var/log/syslog` with the help of `grep` and `tail` commands for the reason of error while connecting to the VM. I could see that ssh serves went down because of multiple failed attempts of authentication. Thats why I was not able to access the VM. And when you restarted the VM the server and port 22 opened again and thats when I was able to connect again.

this is my finding. please let me know if there is anything else as well.

# LW-Second-Task
Introduce:
- Grafana Tempo
- OpenTelemetry
- Multi-tier test application which can be used to demonstrate it's tracing ability.

## Step 1: Set up Tempo with Local Storage

- create a Helm chart configuration values file for Tempo with local storage

```
# tempo-values.yaml
tempo:
  repository: grafana/tempo
  tag: latest
  pullPolicy: IfNotPresent

storage:
  trace:
    backend: local
    local:
      path: /var/tempo/traces

persistence:
  enabled: true
  size: 10Gi
  storageClassName: standard
```

- Install Tempo using Helm

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install tempo grafana/tempo -f tempo-values.yaml
```

## Step 2: Deploy OpenTelemetry Collector

- Create an OpenTelemetry Collector configuration `otel-collector.yaml`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-conf
  namespace: default
data:
  otel-collector-config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      batch:
        timeout: 1s
        send_batch_size: 1024

      memory_limiter:
        check_interval: 1s
        limit_mib: 1000
        spike_limit_mib: 200

      k8sattributes:
        auth_type: "serviceAccount"
        passthrough: false
        extract:
          metadata:
            - k8s.pod.name
            - k8s.pod.uid
            - k8s.deployment.name
            - k8s.namespace.name
            - k8s.node.name
          annotations:
            - key: app
              from: pod
          labels:
            - key: app.kubernetes.io/name
              from: pod

    exporters:
      otlp:
        endpoint: "tempo.monitoring.svc.cluster.local:4317"
        tls:
          insecure: true
      
      logging:
        loglevel: debug
        sampling_initial: 5
        sampling_thereafter: 200

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, k8sattributes, batch]
          exporters: [otlp, logging]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  namespace: default
spec:
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
      - name: collector
        image: otel/opentelemetry-collector-contrib:latest
        args:
        - "--config=/conf/otel-collector-config.yaml"
        ports:
        - containerPort: 4317 # OTLP gRPC
        - containerPort: 4318 # OTLP HTTP
        volumeMounts:
        - name: otel-collector-config-vol
          mountPath: /conf
      volumes:
        - name: otel-collector-config-vol
          configMap:
            name: otel-collector-conf
---
apiVersion: v1
kind: Service
metadata:
  name: otel-collector
  namespace: default
spec:
  ports:
  - name: otlp-grpc
    port: 4317
    targetPort: 4317
  - name: otlp-http
    port: 4318
    targetPort: 4318
  selector:
    app: otel-collector
```

- Apply and deploy the configuration

```
kubectl apply -f otel-collector.yaml
```

## Error faced after this step

otel-collector pod was in CrashBackLoopOff state and was not runnning.

## Finding the Issue that was causing the error
I checked the logs of the deployment

```
kubectl logs deploy/otel-collector
```

