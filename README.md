Blue-Green Deployment Using Argo CD
This guide demonstrates how to implement a Blue-Green Deployment strategy using Argo CD. Blue-Green Deployment involves maintaining two environments (blue and green) and 
switching traffic between them to ensure zero-downtime updates.

Prerequisites
A Kubernetes cluster (e.g., Minikube).
kubectl

Installed minikube on my windows machine (i.e Need to install docker prior as you need a driver to start minikube).
          
minikube start --driver = docker

Step 1: Set Up Your Repository
Create a GitHub repository and build the following structure:
Below is the structure
📂 argocd-blue-green/
├── blue-deployment.yaml
├── green-deployment.yaml 
└── service.yaml

Step 2: Define Blue and Green Deployments
   Create the deployment manifests for the blue and green environments.
   Blue Deployment (blue/deployment.yaml)
   Green Deployment (green/deployment.yaml)

Step 3: Define the Service.yaml file

Step-4 :Need to install the argocd using the below commands.
  
  kubectl create namespace argocd
  
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

  kubectl port-forward service/argocd-server -n argocd 8080:443

Access the Argo UI :
  Open the Argo CD UI in your browser (usually at http://localhost:8080 or the external IP if exposed).

  Log in with your credentials (default username is admin, and the password can be retrieved using the command below):

       kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Create a New Application
  In the Argo CD UI, click on the "New App" button.

  Fill in the following details:

  General Section

  Application Name: blue-green-app

  Project: default

  Sync Policy: Select Automated (optional, but recommended for automatic syncing).

  Source Section
     
     Repository URL: https://github.com/your-repo/argocd-blue-green.git (replace with your repository URL).

  Revision: HEAD (or specify a branch/tag).

  Path: . (or the path to your manifests, e.g., argocd-blue-green/apps).

  Destination Section

  Cluster URL: https://kubernetes.default.svc (in-cluster).

  Namespace: default.

  Sync Options

  Enable Prune (to delete resources that are no longer in the Git repo).

  Enable Self Heal (to automatically correct drifts).

  Enable Create Namespace (if the namespace does not exist).

  Click Create to create the application.

  Verify the Application

  After creating the application, you will be redirected to the application dashboard.

  Wait for Argo CD to sync the application. You should see the Sync Status change to Synced.

  Check the Resource Tree to ensure that all resources (e.g., blue-nginx, green-nginx, and nginx-service) are created and healthy.

Switch Traffic Between Blue and Green
   
   To switch traffic from blue to green, follow these steps:

   Update the service.yaml file in your Git repository:

  selector:
    app: nginx
    version: green  # Switch to green
  
Commit and push the changes to your Git repository.

Argo CD will automatically detect the changes and sync the application. Alternatively, you can manually trigger a sync:

Go to the Argo CD UI.

Select your application (blue-green-app).

Click Sync.

Verify the switch by checking the service endpoints:

  kubectl get endpoints nginx-service -n default




