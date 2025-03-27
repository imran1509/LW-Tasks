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


