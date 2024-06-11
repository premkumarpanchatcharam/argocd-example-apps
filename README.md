# argocd-Installation

Naming convention for ArgoCD & Application deployment GKE Clusters

  a. ArgoCD GEK Cluster: /"<env>/"-argocd-cluster
  
  b. Application Deployment GKE Cluster : '<env>'-app-cluster

A. Create Two GKE Cluster one for ArgoCD and another for Application deployment.
  
  i) Launch Cloud shell and set the env variables:
  
  export PROJECT_ID=\<PROJECT_ID>\
  
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

  iv) Rename the contexts
  
      gcloud container clusters get-credentials prod-argocd-cluster --zone $ZONE --project $PROJECT_ID
      
      gcloud container clusters get-credentials prod-app-cluster --zone $ZONE --project $PROJECT_ID
     
      kubectl config get-contexts --output="name"
 
      kubectl config rename-context \
         gke_<project-id>_<zone>_prod-argocd-cluster prod-argocd-cluster

     kubectl config rename-context \
        gke_<project-id>_<zone>_prod-app-cluster prod-app-cluster
        
B. Installing ArgoCD Application on prod-argocd-cluster:
 
1. Create a namespace "argocd" in the prod-argocd-cluster GKE cluster
      
    kubectl create namespace argocd
     
2. install ArgoCD in the namespace 
 
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
 
3. Install argocd client to connect to argocd server
 
     curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
 
     sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
 
     rm argocd-linux-amd64
 
4. Update the ArgoCD service from Cluster to Loadbalancer to access from internet
 
     kubectl patch svc argocd-server -n argocd --type='json' -p '[{"op":"replace","path":"/spec/type","value":"LoadBalancer"}]'
  
5. get the argocd server IP to login
 
     kubectl --context prod-argocd-cluster \
       --namespace argocd get svc argocd-server -o jsonpath='{.status.loadBalancer}'|
 
6.  get the initial admin password for ArgoCD to login
    
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
 
             or 
 
      argocd admin initial-password -n argocd
 
7. Login into ArgoCD
  
     argocd login <argocd-server IP>

8. Change the admin password

	  argocd account update-password

9. Add the Application GKE Cluster into ArgoCD to deploy applications 
 
     argocd cluster add <cluster-name>

     argocd cluster list
 
10. List the list of accounts created
 
     argocd account list
 
11. List the applications configured
 
     argocd app list

12. Create user and update password on argocd

    kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"accounts.prem":"apikey,login"}}'

    argocd account list

    argocd account update-password --account prem --current-password 'Test@123' --new-password  mysecurepass
 
13. Logout from admin to login as prem

    argocd logout  /"<argocd-server IP>"/
    argocd login <argocd-server IP> --username prem --password mysecurepass
   
C. Configure the Application on ArgoCD for Continuous Delivery.
