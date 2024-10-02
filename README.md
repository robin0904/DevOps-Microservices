Kubernetes Multi-Cluster Deployment with ArgoCD
This guide demonstrates how to deploy and manage multiple Amazon EKS clusters using eksctl and ArgoCD for GitOps-based continuous deployment.

**Step 1: Create EKS Clusters**

Use eksctl to create a hub and two spoke clusters in the same region.

```
eksctl create cluster --name hub-cluster --region ap-south-1
eksctl create cluster --name spoke-cluster-1 --region ap-south-1
eksctl create cluster --name spoke-cluster-2 --region ap-south-1
```

**Step 2: Configure Kubernetes Context**

Check the available Kubernetes contexts and switch to the hub cluster.

```
kubectl config get-contexts | grep ap-south-1
kubectl config use-context <Use-your-contect-uri>
```

Verify the current context:
```
kubectl config current-context
```

**Step 3: Install ArgoCD**

Follow the official ArgoCD documentation to install ArgoCD. The steps are summarized below:

1. Create the argocd namespace:
```
kubectl create namespace argocd
```
2. Apply the ArgoCD manifest to install it:
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
3. Verify the installation:
```
kubectl get pods -n argocd
kubectl get svc -n argocd
```

Step 4: Enable Insecure Mode for ArgoCD
To enable insecure access for testing, update the ArgoCD ConfigMap:
```
kubectl edit configmap argocd-cmd-params-cm -n argocd
```
Add the following to the ConfigMap:
```
data:
  server.insecure: "true"
```
Verify the change:
```
kubectl get pods -n argocd
kubectl get svc -n argocd
```

Step 5: Expose ArgoCD Server with NodePort
Edit the ArgoCD server service to change the type to NodePort:
```
kubectl edit svc argocd-server -n argocd
```
After modifying the service, verify the updated service details:
```
kubectl get svc argocd-server -n argocd
```
Update your EC2 security group to allow traffic on the newly exposed NodePort. You can now access the ArgoCD web UI using the public IP of the EC2 instance and the NodePort.

**Step 6: Access ArgoCD Web UI**

To retrieve the initial admin password for ArgoCD:
1. Get the secret containing the ArgoCD password:
   ```
   kubectl get secrets -n argocd
   ```
2. Edit the secret to find the password:
   ```
   kubectl edit secret argocd-initial-admin-secret -n argocd
   ```
3. Decode the base64-encoded password:
   ```
   echo <Enter your password> | base64 --decode
   ```
   The output will provide the ArgoCD admin password.
   
4. Login to the ArgoCD web UI using admin as the username and the decoded password.

**Step 7: Add EKS Clusters to ArgoCD**

To manage multiple clusters using ArgoCD, add your EKS clusters:
  1. Install the ArgoCD CLI:
     ```
     brew install argocd
     ```
  2. Log in to the ArgoCD server:
     ```
     argocd login <ARGOCD_SERVER_IP>:<NODEPORT>
     ```
     Provide the admin username and the password obtained earlier.
  3. Add the spoke clusters:
     ```
     argocd cluster add terra@spoke-cluster-1.ap-south-1.eksctl.io --server <ARGOCD_SERVER_IP>:<NODEPORT>
     argocd cluster add terra@spoke-cluster-2.ap-south-1.eksctl.io --server <ARGOCD_SERVER_IP>:<NODEPORT>
     ```
  4. Check the web UI to verify that all three clusters are connected.

**Step 8: Deploy Applications via ArgoCD**

To deploy applications to your clusters, follow these steps:

  1. Clone or download the Git repository containing your Kubernetes manifests.
  2. Add the repository to ArgoCD via the web UI.
  3. Sync the repository with the desired cluster using the ArgoCD web UI.

**Step 9: Verify Application Status**

To check the status of your deployments on each cluster, switch between contexts and run the following commands:
```
kubectl config use-context terra@hub-cluster.ap-south-1.eksctl.io
kubectl get all

kubectl config use-context terra@spoke-cluster-1.ap-south-1.eksctl.io
kubectl get all

kubectl config use-context terra@spoke-cluster-2.ap-south-1.eksctl.io
kubectl get all
```
 
