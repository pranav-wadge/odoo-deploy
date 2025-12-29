Odoo 17 on Kubernetes (Simple & Clean Setup)

This repository deploys Odoo 17 with PostgreSQL on Kubernetes using only two YAML files.

postgres.yaml → PostgreSQL + Persistent Storage

odoo.yaml → Odoo Application

No Helm.
No ConfigMaps.
No magic.
Just clean Kubernetes.

📦 Prerequisites

Kubernetes cluster (k3s / minikube / kubeadm)

kubectl configured

Node access (for NodePort)

Docker images:

postgres:16

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
Persistent Volume (PV + PVC)


PostgreSQL data is persistent

Odoo is stateless

Database initialization is done once

🚀 Step 1: Deploy PostgreSQL

Apply PostgreSQL resources:

kubectl apply -f postgres.yaml


Wait until PostgreSQL is running:

kubectl get pods


Expected output:

postgres-xxxxx   1/1   Running


Check storage and service:

kubectl get pvc
kubectl get svc postgres

⚠️ Step 2: Initialize Odoo Database (RUN ONCE)

❗ IMPORTANT
This step is required only once when the database is new.
❌ Do NOT run this on every restart.

Why this is needed

PostgreSQL creates an empty database

Odoo requires its core tables (base module)

This command initializes them

Stop Odoo (if running)
kubectl scale deploy odoo --replicas=0

Initialize the database
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

✅ Expected Result

Logs show base module loaded

Pod exits automatically

Database is now ready

🚀 Step 3: Deploy Odoo
kubectl apply -f odoo.yaml


Check pod status:

kubectl get pods


Expected:

odoo-xxxxx   1/1   Running

🌐 Step 4: Access Odoo

Get Node IP:

kubectl get nodes -o wide


Get NodePort:

kubectl get svc odoo


Example output:

odoo   NodePort   10.x.x.x   8069:30158/TCP


Open in browser:

http://<NODE-IP>:<NODEPORT>


Example:

http://172.16.17.131:30158

🔐 Step 5: Login to Odoo

Use the admin email and password you created in the Odoo UI.

⚠️ These are Odoo credentials, not PostgreSQL credentials.

🔁 Reset Admin Password (From Kubernetes)

If you forget the admin password, you can reset it safely.

Connect to PostgreSQL inside the cluster
kubectl exec -it deploy/postgres -- psql -U odoo odoo

Run this SQL command
UPDATE res_users SET password='admin' WHERE login='admin';


Exit psql:

\q


Now you can log in with:

Email: admin
Password: admin


(You should change it immediately after login.)

🛑 How to Stop Services (Safe)

Stops pods without deleting data:

kubectl scale deploy odoo postgres --replicas=0

🔁 How to Start Again
kubectl scale deploy postgres odoo --replicas=1


👉 No database initialization needed again.

🧠 Notes & Best Practices

Do NOT rerun odoo-init unless the database is deleted

Do NOT change PostgreSQL major version with existing data

For production:

Use StatefulSet for PostgreSQL

Use Longhorn or CSI storage

Use Ingress + TLS

✅ Status

✔ PostgreSQL persistent
✔ Odoo running
✔ Database initialized
✔ Login working

🎉 Done

Your Odoo 17 Kubernetes deployment is complete and clean.