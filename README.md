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

## Step 1 SSH into VM
Connect to the VM using this command in your terminal

```
ssh -i /Users/imran/LWSSHkey root@64.227.139.218
```
