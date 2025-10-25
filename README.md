# Mastering Multi-Cluster Deployments: A GitOps Journey with ArgoCD

<img width="718" height="470" alt="Screenshot 2025-10-25 at 5 59 10 PM" src="https://github.com/user-attachments/assets/c6d6054a-f0d7-4a1a-84e6-5eb5762288c1" />

## Why Deployment with GitOps
In traditional deployment workflows, we typically use tools like Jenkins for Continuous Integration (CI) and rely on shell scripts or Ansible Playbooks for Continuous Delivery (CD) to deploy applications on Kubernetes. While this approach is functional, there are some challenges and limitations.

When considering future updates or version deployments, making changes directly to Kubernetes manifests or configuration files poses a few problems. Firstly, there’s no inherent tracking mechanism to identify who made specific changes. Additionally, if someone updates the Kubernetes resources directly without updating the associated manifest files, it creates a potential disconnect.

To address these issues and enhance the deployment process, the GitOps approach comes into play. GitOps leverages version control systems like Git to manage and track changes to the Kubernetes manifests. Instead of manually editing configuration files or making direct changes to Kubernetes, all modifications are done through Git repositories.

By adopting GitOps, you gain several benefits:

- Traceability and Accountability: GitOps ensures a clear audit trail of changes. Every modification is recorded in the version control system, making it easy to trace back and identify who made specific updates and when.
- Versioning and Rollbacks: With GitOps, you can easily roll back to previous versions of your application by reverting changes in the Git repository. This provides a safety net in case of issues with a new deployment.
- Collaboration: GitOps promotes collaboration by allowing multiple team members to work on the same codebase using Git workflows. This ensures that changes are reviewed, tested, and approved before being applied to the Kubernetes cluster.
- Consistency: All changes, regardless of the environment or contributor, go through the same process. This consistency helps avoid configuration drift and ensures that what is defined in the Git repository accurately represents the state of the deployed application.
- Automation: GitOps integrates seamlessly with CI/CD pipelines, automating the deployment process. Changes pushed to the Git repository trigger automated updates to the Kubernetes cluster, streamlining the entire

### ArgoCD

In our project, we’ve chosen ArgoCD as our GitOps tool among various options like Flux CD and others. ArgoCD plays a crucial role in automating our deployment workflow.

ArgoCD essentially acts as a synchronization engine between your GitHub repository and the Kubernetes cluster. It constantly monitors the specified GitHub repository for any changes. Whenever a change is detected, ArgoCD takes charge and applies those modifications to the connected Kubernetes cluster.

Behind the scenes, ArgoCD’s architecture includes a Kubernetes controller responsible for keeping an eye on the Git repository. It operates under the principle that the Git repository serves as the single source of truth for your application configuration. Any manual changes made directly to the Kubernetes cluster are automatically identified and rolled back by the controller, ensuring that the desired configuration specified in the Git repository is always maintained.

ArgoCD offers a range of powerful capabilities. Here’s a breakdown:
 - Tracking: ArgoCD keeps a close track of changes happening in your Git repository, providing transparency into the evolution of your application configuratio
 - Auditing: The tool maintains an audit trail, helping you understand who made specific changes and when they were applied.
 - Monitoring: ArgoCD actively monitors the Git repository for updates, ensuring that your Kubernetes cluster is always in sync with the latest version of your application configuration.
 - Revoking Changes: If someone manually alters the configuration in the Kubernetes cluster, ArgoCD automatically detects and reverts those changes to align with the version defined in the Git repository.
 - Auto-healing: ArgoCD contributes to the resilience of your application by automatically correcting any discrepancies between the desired state in the Git repository and the actual state in the Kubernetes cluster.

## Now Let’s Begin with building Project
Some of the Prerequisite that you need to download before we start implementing the Project
   - kubectl — A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl. https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
   - eksctl — A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating. https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
   - AWS CLI — A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html in the AWS Command Line Interface User Guide.
   - After installing the AWS CLI, I recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide. https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config
   - Argo CD CLI(Not the actual Argo CD Installation) — https://argo-cd.readthedocs.io/en/stable/cli_installation/#installation

Now that we’ve set up the prerequisites, let’s move on to implementing the project. Our first step is to create three separate clusters. To achieve this, we’ll utilize the eksctl command with the following commands:
```bash
  eksctl create cluster --name hub-cluster --region us-east-1
  eksctl create cluster --name spoke-cluster-1 --region us-east-1
  eksctl create cluster --name spoke-cluster-2 --region us-east-1
```

<img width="1554" height="327" alt="Screenshot 2025-10-25 at 5 04 57 PM" src="https://github.com/user-attachments/assets/b24dbd85-619c-48f6-ac4e-8081b220645e" />

1. Argo CD offers two primary deployment models: the Standalone model and the Hub-Spoke model, both serving distinct purposes in managing Kubernetes clusters.
Standalone Model: In the Standalone model, every Kubernetes cluster operates independently with its own instance of Argo CD. Each Argo CD instance is configured to monitor a specific GitOps repository. When changes are detected in the repository, the corresponding Argo CD instance takes charge and applies those changes to its associated Kubernetes cluster. Essentially, each Argo CD instance is self-contained and responsible for managing its respective Kubernetes environment.

2. Hub-Spoke Model: In the Hub-Spoke model, a centralized hub cluster is designated to host the Argo CD instance. This hub cluster serves as the control center and is responsible for managing resources across multiple spoke clusters. The spoke clusters, which could be numerous (e.g., 5 or 10), rely on the hub cluster for coordination and deployment. The Argo CD instance in the hub cluster is configured to oversee and make changes to the other clusters, offering a centralized and streamlined approach to cluster management.

This model is particularly useful in scenarios where there’s a need for centralized control and coordination across a network of clusters. The hub cluster ensures consistency in application deployments, versioning, and configuration management across all connected spoke clusters.

It’s worth noting that while these are the main deployment models, Argo CD is flexible, allowing for various configurations to suit specific project requirements. The choice between the Standalone and Hub-Spoke models depends on factors like the scale of the deployment, the level of centralized control needed, and the overall architecture preferences of the project.

Once you have successfully created all the clusters in your Amazon EKS environment using the provided commands, you can verify the Kubernetes resources within these clusters by using the following command:

```bash
  kubectl config get-contexts
```
This command provides an overview of available Kubernetes contexts, each associated with a specific cluster. If you want to narrow down the list based on specific details, such as the region, you can use the grep command. For example:
```bash
  kubectl config get-contexts | grep us-east-1
```
This command filters the list to show only the clusters created in the “us-east-1” region.

By default, your kubectl commands will apply to the cluster that was created most recently. To switch to a different cluster, you can use the following command, specifying the desired context:
```bash
  kubectl config use-context iam-root-account@hub-cluster.us-east-1.eksctl.io
```
This command changes the active context to the specified cluster, allowing you to interact with and manage resources in that particular Kubernetes cluster. Adjust the context as needed based on your project requirements or the specific cluster you want to work with.

<img width="1019" height="63" alt="Screenshot 2025-10-25 at 5 11 18 PM" src="https://github.com/user-attachments/assets/e3e50adb-89fb-4e8d-9a41-7b9152377e6e" />

To install ArgoCD on your hub-spoke cluster, follow these simple commands:
```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
<img width="1059" height="263" alt="Screenshot 2025-10-25 at 5 12 06 PM" src="https://github.com/user-attachments/assets/ebefe9a3-f4e2-4408-8a35-960c66ae363f" />

These commands create the necessary namespace for ArgoCD and apply the installation manifest from the official ArgoCD documentation. This straightforward process sets up ArgoCD on your hub-spoke cluster, paving the way for the next steps in the project.

To verify that ArgoCD is up and running
```bash
  kubectl get pods -n argocd
```
<img width="1054" height="199" alt="Screenshot 2025-10-25 at 5 12 45 PM" src="https://github.com/user-attachments/assets/80f37f1e-eb4b-4f8e-9d07-ba4b89c3b235" />

For the purpose of this project demo, we will run the ArgoCD API server in insecure mode. Typically, in real-time projects, we opt for the secure mode using HTTPS. However, for the sake of simplicity and a quicker demonstration, we’ll utilize HTTP mode in this project. Let’s proceed to configure it.
```bash
  kubectl get cm -n argocd
```
<img width="1057" height="230" alt="Screenshot 2025-10-25 at 5 14 08 PM" src="https://github.com/user-attachments/assets/652ed5cd-9540-4571-b092-632ffbf63285" />

To configure ArgoCD to run in HTTP mode, follow these steps:

1. Retrieve all configmaps in ArgoCD:
  - kubectl get configmaps -n argocd
  - Locate the argocd-cmd-params-cm configmap.
  - Edit the configmap to enable HTTP mode by adding the following data:
  - yamlCopy code
    ```yaml
    data:
      server.insecure: "true"
    ```
<img width="1394" height="381" alt="Screenshot 2025-10-25 at 6 19 47 PM" src="https://github.com/user-attachments/assets/60133256-6d91-4f41-b9b1-f2d4d379badf" />

This change allows ArgoCD to run in HTTP mode. Ensure to save the modifications. This adjustment is typically done in the argocd-cmd-params-cm configmap within the ArgoCD namespace.

Now all of our pod and services are running let’s verify using:
```bash
  kubectl get svc -n argocd
```
<img width="1057" height="356" alt="Screenshot 2025-10-25 at 5 15 37 PM" src="https://github.com/user-attachments/assets/33dc7d8e-8d0a-4e52-b8c1-498503241b4b" />
  - This command lists all the running services in the ArgoCD namespace
  - For this project, we’ll use NodePort mode to access the application. To enable this, make a simple adjustment to the argocd-server service
  - Edit the argocd-server service file.
  - Change the type from ClusterIP to NodePort.
  - Save the file.

<img width="424" height="114" alt="Screenshot 2025-10-25 at 5 16 47 PM" src="https://github.com/user-attachments/assets/3283f7ad-dbf4-41a6-868f-84aea56a1a09" />

This change allows you to access ArgoCD using the NodePort mode, facilitating external access to the application. Adjust the service type and save the file accordingly.

<img width="1063" height="344" alt="Screenshot 2025-10-25 at 5 17 22 PM" src="https://github.com/user-attachments/assets/7335c7ae-858f-46aa-bfca-d609e4606846" />
### To access the ArgoCD application in NodePort mode:
1. Identify the hub-spoke instance created when the cluster was set up.
2. Obtain the public IP address of the hub-spoke instance.
3. Use the public IP address along with the NodePort specified in the argocd-server service to access ArgoCD.
4. Example: http://<public-ip>:<NodePort>

### Additionally, for external access through HTTP and the internet:
  - Modify the security group associated with the hub-spoke instance.
  - Allow incoming traffic on the NodePort specified in the argocd-server service.
    
<img width="1590" height="280" alt="Screenshot 2025-10-25 at 5 19 35 PM" src="https://github.com/user-attachments/assets/3b3707ec-e3a3-4859-a4a9-ab280ca0aaa1" />

This adjustment enables access to ArgoCD via HTTP from external sources. Ensure the security group rules are updated to permit the desired traffic for seamless.
Now you’ll have an access of the ArgoCD user interface by following the above steps: It will look like this

<img width="1901" height="1021" alt="Screenshot 2025-10-25 at 5 20 09 PM" src="https://github.com/user-attachments/assets/851760c8-c3c6-4dbc-9369-851a3941c886" />

Retrieve the initial admin password stored in a secret file:
```bash
  kubectl get secrets -n argocd
```
<img width="1045" height="170" alt="Screenshot 2025-10-25 at 5 20 42 PM" src="https://github.com/user-attachments/assets/bb419198-65f9-4caa-bd40-292ca4f4e087" />

- Identify the argocd-initial-admin-secret file.
- Access the secret file to obtain the encoded password:
```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
```

Locate the password within the file and copy it.
Decode the password using the following command:
echo <encoded-password> | base64 --decode
Replace <encoded-password> with the copied password.
Copy the decoded password and paste it into the password field on the ArgoCD user interface.
