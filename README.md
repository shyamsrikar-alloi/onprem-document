# Onprem-Document Performed-Tests
## Cloned in to the particular repo
```
git clone --branch INFRA-11-restructure-alloi-stack-and-dependencies https://github.com/opshealth/alloi-public-charts.git
```
- username: shyamsrikar-alloi
- password: (token-classic)
# Step 1: Install Dependencies

- Install cert-manager for SSL certificates
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml 
```
- Install NGINX ingress controller
``` 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml 
```
- Wait for services to be ready
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

# Step 2: Create SSL Certificate Issuer
```
cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: shyamsrikar@alloi.ai  
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```
<img width="1135" height="405" alt="image" src="https://github.com/user-attachments/assets/0e8b6413-f1b1-43bb-8093-0bbeb1c04fd9" />

# Step 3: Set Up PostgreSQL Databases
### Connect to your PostgreSQL instance and create the required databases:

```
-- Create databases
CREATE DATABASE "alloi-embeddings";
CREATE DATABASE "alloi-jackson";
CREATE DATABASE "alloi-supertokens";
CREATE DATABASE "alloi-backend";
```
```
-- Create users
CREATE USER embeddings_user WITH PASSWORD 'secure_password_1';
CREATE USER jackson_user WITH PASSWORD 'secure_password_2';
CREATE USER supertokens_user WITH PASSWORD 'secure_password_3';
CREATE USER backend_user WITH PASSWORD 'secure_password_4';
```
```
-- Grant permissions
GRANT ALL PRIVILEGES ON DATABASE "alloi-embeddings" TO embeddings_user;
GRANT ALL PRIVILEGES ON DATABASE "alloi-jackson" TO jackson_user;
GRANT ALL PRIVILEGES ON DATABASE "alloi-supertokens" TO supertokens_user;
GRANT ALL PRIVILEGES ON DATABASE "alloi-backend" TO backend_user;
```

# Step 4: Clone and Prepare Alloi Charts

- Clone the repository
```
# git clone https://github.com/opshealth/alloi-public-charts.git
git clone --branch INFRA-11-restructure-alloi-stack-and-dependencies https://github.com/opshealth/alloi-public-charts.git
```
```
cd alloi-public-charts
```
- Update chart dependencies
```
helm dependency update ./alloi-stack
```
- Create namespace
```
kubectl create namespace alloi
```

# Step 5: Configure DNS
- Get your load balancer IP/hostname
```
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

- Point your domain to the load balancer
- Create DNS A record: alloi.yourdomain.com -> [LOAD_BALANCER_IP]
```
sudo nano /etc/hosts
```
- Added this line

```
127.0.0.1   alloi.localhost
```




# Step 6: Configure values.yaml
Edit ./alloi-stack/values.yaml with your settings:

<img width="1764" height="865" alt="image" src="https://github.com/user-attachments/assets/984cb07c-3e3f-4283-878f-d146e1d768ca" />
