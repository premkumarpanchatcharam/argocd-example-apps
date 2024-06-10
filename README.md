# argocd-Installation

Naming convention for ArgoCD & Application deployment GKE Clusters
  a. ArgoCD GEK Cluster: <env>-argocd-cluster
  b. Application Deployment GKE Cluster : <env>-app-cluster

A. Create Two GKE Cluster one for ArgoCD and another for Application deployment.
  
  i) Launch Cloud shell and set the env variables:
  export PROJECT_ID=<PROJECT_ID>
  export ZONE=<ZONE>

  ii) Create the ArgoCD GKE Cluster
   gcloud container clusters create prod-argocd-cluster \
    --project PROJECT_ID \
    --zone=ZONE \
    --machine-type "n1-standard-1" \
    --num-nodes "2" --min-nodes "2" --max-nodes "2" \
    --enable-ip-alias --enable-autoscaling --disk-size 10 --async
  iii) Create the Application GKE Cluster
   gcloud container clusters create prod-app-cluster \
    --project PROJECT_ID \
    --zone=ZONE \
    --machine-type "n1-standard-1" \
    --num-nodes "2" --min-nodes "2" --max-nodes "2" \
    --enable-ip-alias --enable-autoscaling --disk-size 10 --async
B. Installing ArgoCD Application on prod-argocd-cluster:
   
C. Configuring the GKE Cluster for App deployment.
