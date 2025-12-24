Odoo 17 on Kubernetes (Simple & Clean Setup)

This repository deploys Odoo 17 with PostgreSQL on Kubernetes using only two YAML files.

postgres.yaml → Database + Storage

odoo.yaml → Odoo Application

No Helm, no ConfigMaps, no confusion.

📦 Prerequisites

Kubernetes cluster (k3s / minikube / kubeadm)

kubectl configured

Node access (for NodePort)

Docker images available:

postgres:14

odoo:17

📁 Repository Structure
.
├── postgres.yaml
├── odoo.yaml
└── README.md

🧠 Architecture Overview
Odoo Pod
   |
   v
PostgreSQL Service
   |
   v
PostgreSQL Pod
   |
   v
Persistent Volume (PVC + PV)


PostgreSQL data is persistent

Odoo is stateless

Database initialization is done once

🚀 Step 1: Deploy PostgreSQL
kubectl apply -f postgres.yaml


Wait until PostgreSQL is running:

kubectl get pods


Expected:

postgres-xxxxx   1/1   Running


Check services and storage:

kubectl get svc
kubectl get pvc

⚠️ Step 2: Initialize Database (RUN ONCE)

IMPORTANT

This step is required only once per database.

❌ Do NOT run this on every restart
✅ Run only when the database is new or empty

Why this is needed

PostgreSQL creates an empty database

Odoo requires its core tables (base module)

This step initializes them

⛔ Stop Odoo before initialization (if running)
kubectl scale deploy odoo --replicas=0

▶️ Initialize Odoo database
kubectl run odoo-init \
  --rm -it \
  --image=odoo:17 \
  --restart=Never \
  --command -- \
  odoo \
    -d odoo \
    -i base \
    --db_host=postgres \
    --db_port=5432 \
    --db_user=odoo \
    --db_password=odoo \
    --stop-after-init

✅ Expected result

Logs show Module base loaded

Pod exits automatically

Database is now ready

🚀 Step 3: Deploy Odoo
kubectl apply -f odoo.yaml


Wait until Odoo is running:

kubectl get pods


Expected:

odoo-xxxxx   1/1   Running

🌐 Step 4: Access Odoo

Get Node IP:

kubectl get nodes -o wide


Get NodePort:

kubectl get svc odoo


Example output:

odoo   NodePort   10.x.x.x   8069:31233/TCP


Open in browser:

http://<NODE-IP>:<NODEPORT>


Example:

http://172.16.17.131:31233

🔐 Step 5: Login to Odoo

Email: the admin email you set

Password: the admin password you set

⚠️ These are Odoo credentials, not PostgreSQL credentials.

🛑 How to Stop Services (Safe)

Stops pods but keeps data:

kubectl scale deploy odoo --replicas=0
kubectl scale deploy postgres --replicas=0

🔁 How to Start Again
kubectl scale deploy postgres --replicas=1
kubectl scale deploy odoo --replicas=1


👉 No database initialization needed again.