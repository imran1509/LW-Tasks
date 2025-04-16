# Everyday Summary

As per Callum's suggestion, I will be documenting my daily summaries here moving forward. Additionally, I will provide brief combined summary for the past days. Starting from today, March 27th, I will structure my summaries to include what I attempted, what I achieved, the challenges I encountered, and how I worked to overcome them and find solutions.

## :date: 17 March - 26 March

I joined LiveWyer's slack on 17th March and got my first task on 18th March

In my first task, I SSH'd into a VM and set up a local Kubernetes cluster using K3s. I installed essential tools, including Neovim, Zsh, Git, Kubectl, Terraform, and Helm. I then cloned the Bitnami charts repository via Git, edited the values.yaml file of the Nginx chart using Neovim to set replicaCount to 2, and deployed it to the local Kubernetes cluster using Helm. Finally, I verified the deployment using Kubectl to ensure two replicas were running successfully. 

Following this, the next day, I received an invitation to Tailscale from David. He provided me with a new VM address and credentials, requesting me to repeat the same task on the new VM.

After setting up the Kubernetes cluster, for my next task David instructed me to select an open-source observability stack, deploy it, and provide details on the chosen stack along with the reasoning behind my selection.

I selected Prometheus and Grafana as my observability stack and deployed them to collect metrics and visualize them through a Grafana dashboard.

I chose this solution because it is lightweight, scalable, and well-integrated with Kubernetes. Prometheus efficiently gathers and stores real-time metrics while offering built-in alerting via Alertmanager. Grafana enables intuitive visualization with customizable dashboards and supports multiple data sources. This combination is ideal for monitoring Kubernetes clusters, diagnosing issues, and setting up alerts, all while maintaining cost efficiency.

Following this, David assigned me the next task, which involved enhancing the observability stack by introducing Grafana Tempo and OpenTelemetry. Additionally, I was required to deploy a multi-tier test application to demonstrate its tracing capabilities.

I installed Grafana Tempo with local storage as object storage and OpenTelemetry collector using helm. 

I faced an error after OpenTelemetry collector was deployed. otel-collector pod was in CrashBackLoopOff state and was not runnning. I checked the logs of the deployment. The Issue was in the Configuration file. One of the exporters used logging was deprecated. Thats why pod was crashing and not runnning. We have to use `debug` exporter instead of `logging` exporter in the configuration file. To resolve this, I updated the configuration to use the debug exporter instead of logging, restarted the pod, and successfully restored its functionality.


## :date: 27 March - 2 April

I tried to do manual instrumentation but I was having alot of issues then I chose that I will use tempo with local storage and Open telemetry with auto instrumentation and open telemetry operator.

I installed and configured both  the tools and completed the task. I faced many errors as I have mentioned in the main Readme file.

## :date: 3 April - 7 April

After completing the task, I was not able to get the traces in my grafana dashboard. I tried multiple methods to fix the issue as mentioned in the main readme file. I found out that the collector is not sending or receiving ant traces.

IN this whole task I also learned that I need to focus on my coding more. Because manual instrumentation will be much more useful in real time project to get very specific metrics and traces. So I started learning coding with Golang.

## :date: 8 April - 10 April

I had a call with David and Callum to provide a task update and discuss the next steps. The following objective was to debug the entire application and ensure that traces were successfully collected and visualized in the Grafana dashboard. Additionally, I was instructed to re-create the full setup using GitOps with ArgoCD, as the current installation was not stored or versioned, making it non-reproducible in the event of a namespace deletion.

During this period, I focused on debugging the application and was able to successfully get the traces displayed in the Grafana dashboard.

## :date: 11 April - 16 April
I created a dedicated repository for the GitOps task and began setting up the environment accordingly. I successfully deployed the complete application onto the ArgoCD server, though I encountered several errors and challenges throughout the process. Currently, both the application and the observability stack are running as expected. However, I am still working on resolving an issue where traces are not appearing in the Grafana dashboard, and I am actively investigating and addressing this.
