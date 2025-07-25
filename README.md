# Onprem-Document Performed-Tests
Followed this Guide (Alloi On-Premises Deployment - Simple Setup Guide)

# Step 1: Install Dependencies

Install cert-manager for SSL certificates
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml 
```
Install NGINX ingress controller
``` 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml 
```
Wait for services to be ready
``` 
kubectl wait --for=condition=available --timeout=300s deployment -n cert-manager --all 
```
``` 
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx --timeout=300s 
```
Verify the pods 
```
kubectl get pods -n cert-manager
```
```
kubectl get pods -n ingress-nginx
```
<img width="924" height="288" alt="image" src="https://github.com/user-attachments/assets/6a4e0337-4dcd-4aba-ad6e-fa2708b51de5" />

# Step 2: Create SSL Certificate Issuer
Directly runned the below command by modifying email in the script
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
- Connect to your PostgreSQL instance and create the required databases:

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

Clone the repository
```
# git clone https://github.com/opshealth/alloi-public-charts.git
git clone --branch INFRA-11-restructure-alloi-stack-and-dependencies https://github.com/opshealth/alloi-public-charts.git
```
Enter the username and classic-token
- username: shyamsrikar-alloi
- password: (token-classic)

```
cd alloi-public-charts
```
Update chart dependencies
```
helm dependency update ./alloi-stack
```
Create namespace
```
kubectl create namespace alloi
```

# Step 5: Configure DNS
Get your load balancer IP/hostname
```
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

Point your domain to the load balancer
Create DNS A record: alloi.yourdomain.com -> [LOAD_BALANCER_IP]
- I am using my localhost instead od domain so that i have modified the below path
```
sudo nano /etc/hosts
```
Added this line at the end of the data already present in this path

```
127.0.0.1   alloi.localhost
```

# Step 6: Configure values.yaml
Edit ./alloi-stack/values.yaml with your settings:
```
global:
  domain: alloi.localhost
  hostname: localhost
  storage_class: "standard"
  s3_bucket: "local-bucket"
  aws_region: ap-south-1
  database_host: "database-1.c3k6ao6mq5h0.ap-south-1.rds.amazonaws.com"
  database_port: "5432"
```
<img width="1853" height="788" alt="image" src="https://github.com/user-attachments/assets/7a055640-4b38-4f63-951d-868ba22bb592" />

# Step 7: Configure Secrets
Edit ./alloi-stack/values-secrets.yaml with your credentials:
```
DATABASE_NAME: "alloi-backend"
DATABASE_USER: "backend_user"
DATABASE_PASSWORD: "secure_password_4"
```
<img width="1852" height="999" alt="image" src="https://github.com/user-attachments/assets/e43f8bc2-8b12-4f3f-9102-269dd310f499" />

# Generate secure keys:
```
# Django secret key
python3 -c "import secrets; print(secrets.token_urlsafe(50))"

# Cryptography key
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```
# Step 8: Deploy Alloi
Deploy using Helm
```
helm install alloi ./alloi-stack -n alloi \
  -f ./alloi-stack/values.yaml \
  -f ./alloi-stack/values-secrets-template.yaml
```
- Monitor deployment (wait for all pods to be Running)
```
kubectl get pods -n alloi -w
```
<img width="1843" height="982" alt="image" src="https://github.com/user-attachments/assets/09e789b1-ffd8-4d97-bc51-e05f2f03eb91" />
