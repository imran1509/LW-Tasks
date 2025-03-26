# Everyday Summary

As per Callum's suggestion, I will be documenting my daily summaries here moving forward. Additionally, I will provide brief combined summary for the past days. Starting from today, March 26th, I will structure my summaries to include what I attempted, what I achieved, the challenges I encountered, and how I worked to overcome them and find solutions.

## :date: 17 March - 25 March

I joined LiveWyer's slack on 17th March and got my first task on 18th March

In my first, I SSH'd into a VM and set up a local Kubernetes cluster using K3s. I installed essential tools, including Neovim, Zsh, Git, Kubectl, Terraform, and Helm. I then cloned the Bitnami charts repository via Git, edited the values.yaml file of the Nginx chart using Neovim to set replicaCount to 2, and deployed it to the local Kubernetes cluster using Helm. Finally, I verified the deployment using Kubectl to ensure two replicas were running successfully. 

Following this, the next day, I received an invitation to Tailscale from David. He provided me with a new VM address and credentials, requesting me to repeat the same task on the new VM.

After setting up the Kubernetes cluster, for my next task David instructed me to select an open-source observability stack, deploy it, and provide details on the chosen stack along with the reasoning behind my selection.
