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

### :x: Errors and Issues

#### Error 1: Getting error while connecting to the VM

After completing the first task, David shared the credentials for another VM and asked to perform the first task again there. But I was not able to SSH into it. And then Walter and Anthony checked the VM to troubleshoot the issue. Then me, Walter and Anthony tried to find the issue on call.

I checked the following log files `var/log/auth.log` , `var/log/syslog` with the help of `grep` and `tail` commands for the reason of error while connecting to the VM. I could see that ssh serves went down because of multiple failed attempts of authentication. Thats why I was not able to access the VM. And when you restarted the VM the server and port 22 opened again and thats when I was able to connect again.

And i took whole day. In the end we found out that the policies applied by Walter took some time to be efective and then I was able to access the VM.

this is my finding. please let me know if there is anything else as well.

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

- Install the OpenTelemetry Operator to manage OpenTelemetry resources

```
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml
```

### Step 4: Configure OpenTelemetry Collector

- Create an OpenTelemetry Collector configuration file

```
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
  namespace: observability
spec:
  mode: deployment
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      batch:
        timeout: 5s
        send_batch_size: 1000
      memory_limiter:
        check_interval: 1s
        limit_mib: 1000
        spike_limit_mib: 200

    exporters:
      otlp:
        endpoint: tempo:4317
        tls:
          insecure: true

      debug:
        verbosity: detailed

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [otlp, debug]
```

- Apply this configuration

```
kubectl apply -f otel-collector-config.yaml
```

### Step 5: Configure Auto-Instrumentation for Java Services

- Create an instrumentation resource `java-instrumentation for Java services

```
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: java-instrumentation
  namespace: default
spec:
  exporter:
    endpoint: http://otel-collector-collector.observability.svc.cluster.local:4318
  propagators:
    - tracecontext
    - baggage
    - b3
  sampler:
    type: parentbased_traceidratio
    argument: "1.0"
  java:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-java:latest
```

- Apply this configuration

```
kubectl apply -f java-instrumentation.yaml
```

### Step 6: Configure Auto-Instrumentation for Go and Python Services

- Create instrumentation resources for Go and Python services: `go-instrumentation` and `python-instrumentation`

```
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: go-instrumentation
  namespace: default
spec:
  exporter:
    endpoint: http://otel-collector-collector.observability.svc.cluster.local:4318
  propagators:
    - tracecontext
    - baggage
    - b3
  sampler:
    type: parentbased_traceidratio
    argument: "1.0"
  go:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-go:latest
```

```
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: python-instrumentation
  namespace: default
spec:
  exporter:
    endpoint: http://otel-collector-collector.observability.svc.cluster.local:4318
  propagators:
    - tracecontext
    - baggage
    - b3
  sampler:
    type: parentbased_traceidratio
    argument: "1.0"
  python:
    image: ghcr.io/open-telemetry/opentelemetry-operator/autoinstrumentation-python:latest
```

- Apply these configurations

```
kubectl apply -f go-instrumentation.yaml
kubectl apply -f python-instrumentation.yaml
```

### Step 7: Add instrumentaton annotations to all of the service's manifests

For each service in the Online Boutique application, we'll need to edit the original manifest files in the `kubernetes-manifests` directory.

- Java Services: `currencyservice`, `paymentservice`
- Go Services: `productcatalogue`, `frontend`, `checkoutservice`, `shippingservice`
- Python Services: `emailservice`, `recommendationservice`

#### 1. Java Services
For each Java service, manually edit the file to add the following:

#### For example: `currencyservice.yaml`

- Find the `template:` section under the Deployment
- Immediately after `template:`, add:

```
metadata:
    labels:
      app: currencyservice
    annotations:
      instrumentation.opentelemetry.io/inject-java: "java-instrumentation"
```

`Metadata` section will be already there, we just need to add the `annotations` section.

- Find the `containers:` section for the main container
- Add to the `env:` section (create it if it doesn't exist):

```
- name: OTEL_SERVICE_NAME
  value: "currencyservice"
- name: OTEL_RESOURCE_ATTRIBUTES
  value: "service.namespace=default,service.name=currencyservice"
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: "otel-collector-collector.observability.svc.cluster.local:4317"
```

- Similarly modify other Java service manifest's files.

#### 2. Go Services
For Go services, there are is additional annotations that is required:

#### For example: `frontend.yaml`

- Find the `template:` section
- Add after the existing metadata and labels:

```
annotations:
    instrumentation.opentelemetry.io/inject-go: "go-instrumentation"
    instrumentation.opentelemetry.io/otel-go-auto-target-exe: "/src/server"
```

- Add to the `env:` section:

```
- name: OTEL_SERVICE_NAME
  value: "frontend"
- name: OTEL_RESOURCE_ATTRIBUTES
  value: "service.namespace=default,service.name=frontend"
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: "otel-collector-collector.observability.svc.cluster.local:4317"
```

- Similarly modify other Go service manifest's files.

#### 3. Python Services

For the Python service:

#### For example `recommendationservice` 

- Find the `template:` section
- Add after the existing metadata and labels:

```
annotations:
    instrumentation.opentelemetry.io/inject-python: "python-instrumentation"
```

- Add to the `env:` section:

```
- name: OTEL_SERVICE_NAME
  value: "recommendationservice"
- name: OTEL_RESOURCE_ATTRIBUTES
  value: "service.namespace=default,service.name=recommendationservice"
- name: OTEL_EXPORTER_OTLP_ENDPOINT
  value: "otel-collector-collector.observability.svc.cluster.local:4317"
```

### Step 8: Deploy the application

- After manually modifing all the manifests, apply them and deploy the application:

```
kubectl apply -f kubernetes-manifests/
```

### Step 9: Install grafana for visualization

- Add and update Grafana Helm repository and get the Grafana Helm chart.

```
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
```

- Install Grafana

```
helm install grafana grafana/grafana -n observability
```

- To access the Grafana server application on our browser we have to expose the service.

- To expose the service run this command

```
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```

- Access Grafana on browser and login
- Default username will be `admin` and to get the password run the following command

```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

- Login to Grafana and add Tempo as data source

- Then go to Explore and see the traces by service name. 

---

### :bulb: Interesting and new things I learned until these step.
I got to learn about:

#### Traces

- Traces are detailed records of the path that a request or transaction takes as it moves through a system especially in distributed or microservices architectures.

- A trace typically consists of multiple spans.

#### Span

- A span is a single operation within the trace, like a function call, a request to a database, or an HTTP call to another service.

- Together, these spans give us a timeline of events, showing what happened, where, and how long each part took.

#### Traces help answer questions like:

- Where is the system spending the most time?

- Why is this request slower than usual?

- Which service caused the error?


#### Difference between **Traces** , **Metrics** and **Logs**
- These 3 are known as 3 pillars of observability

📊 Traces vs Metrics vs Logs

| Feature   | **Traces**                              | **Metrics**                          | **Logs**                             |
|-----------|------------------------------------------|--------------------------------------|--------------------------------------|
| **What**  | Timeline of a request across services     | Numerical data over time             | Text-based record of events          |
| **Focus** | Request journey (end-to-end flow)        | System health and performance trends | Detailed debug/info/error messages   |
| **Use for** | Debugging distributed systems            | Monitoring, alerting, dashboards     | Troubleshooting, auditing            |
| **Example** | API request from frontend → backend → DB | CPU usage = 80%, Request count = 100 | "Payment failed due to timeout"      |
| **Tools** | Jaeger, Zipkin, Tempo                    | Prometheus, Grafana, CloudWatch      | Loki, ELK, Fluentd, Splunk           |

### :x: Errors and Issues

#### Error 1: Otel-collector pod not running.

After installing Open telemetry collector. When I checked the pods, teh collector pod was not running. It was in CrashBackLoopOff state.

**Steps to find the issue/Cause found** : Then I checked the logs to find the issue. And the cause I found was that in `exporter` section in the the confguration file I had used `logging` as one of the exporter which was deprected.

**Solution** : SO I removed the `loggin` exporter and re-apply the configuration file and deployed the pod again. And the issue was resolved.


#### Error 2: Application pods were not runnning

When I applied all the manifests files after modifying them with annotations. Then the pods were not running and were in ImagePullBackOff state.

**Steps to find the issue/Cause found** : I checked the container image of each deployment by using `kubectl describe` command and I found out that the container image were different and incorrect from the ones mentioned in the manifests.

**Solution** : I set the image for every deployment as the correct container image using `kubectl set image` comand

```
kubectl set image deployment/adservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/adservice:v0.10.2
kubectl set image deployment/cartservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/cartservice:v0.10.2
kubectl set image deployment/checkoutservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/checkoutservice:v0.10.2
kubectl set image deployment/currencyservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/currencyservice:v0.10.2
kubectl set image deployment/emailservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/emailservice:v0.10.2
kubectl set image deployment/loadgenerator main=us-central1-docker.pkg.dev/google-samples/microservices-demo/loadgenerator:v0.10.2
kubectl set image deployment/paymentservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/paymentservice:v0.10.2
kubectl set image deployment/productcatalogservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/productcatalogservice:v0.10.2
kubectl set image deployment/recommendationservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/recommendationservice:v0.10.2
kubectl set image deployment/shippingservice server=us-central1-docker.pkg.dev/google-samples/microservices-demo/shippingservice:v0.10.2
```

#### Error 3: Not getting any traces on Grafana dashboard

After I completed the task and added Tempo as the data source in Grafana dashboard. And I went to explore section to see the traces. I am not able to see any of it.

**Steps to find the issue/Cause found** : 
- Check if OpenTelemetry Collector is running properly.

```
# Check collector pod status
kubectl get pods -n observability -l app.kubernetes.io/component=opentelemetry-collector

# View collector logs for any errors
kubectl logs -l app.kubernetes.io/component=opentelemetry-collector -n observability
```
![](https://github.com/imran1509/LW-Tasks/blob/main/Screenshots/screenshot1.png)

the output shows that the collector is running but but bot sending or receiving any traces.

- Verify Tempo is receiving traces

```
# Check Tempo pod status
kubectl get pods -n observability

# Check Tempo logs
kubectl logs -l app.kubernetes.io/name=tempo -n observability -c tempo
```

These logs show that Tempo is up and running, but they don't show any trace ingestion activity. The logs are only showing regular polling of the blocklist, which is normal background activity.
Since Tempo seems to be running correctly but not receiving traces, let's focus on the connectivity between the OpenTelemetry Collector and Tempo:

- Verify Auto-Instrumentation

First, let's confirm the auto-instrumentation is properly applied to your application pods:

```
kubectl describe pod -n default -l app=frontend
```

Look for instrumentation containers and the annotations

- checked Service Discovery Issues


