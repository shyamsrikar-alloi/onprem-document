# Onprem-Document Performed-Tests
## Cloned in to the particular repo
```
git clone --branch INFRA-11-restructure-alloi-stack-and-dependencies https://github.com/opshealth/alloi-public-charts.git
```
- username: shyamsrikar-alloi
- password: (token-classic)
# Step 1: Install Dependencies

### Install cert-manager for SSL certificates
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml 
```
### Install NGINX ingress controller
``` 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml 
```
### Wait for services to be ready
``` 
kubectl wait --for=condition=available --timeout=300s deployment -n cert-manager --all 
```
``` 
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx --timeout=300s 
```
```
kubectl get pods -n cert-manager
```
```
kubectl get pods -n ingress-nginx
```
<img width="924" height="288" alt="image" src="https://github.com/user-attachments/assets/6a4e0337-4dcd-4aba-ad6e-fa2708b51de5" />
