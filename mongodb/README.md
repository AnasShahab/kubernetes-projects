# mongodb
> This Kubernetes setup deploys a secure MongoDB database alongside the Mongo Express web-based admin interface, enabling easy database management. It includes an Ingress resource that routes external HTTP traffic from mongodb-app.com to the Mongo Express service, allowing convenient browser-based access via Traefik.

## Overview of files:
### Secret (mongodb-secret.yaml)
Stores sensitive credentials such as MongoDB root username/password and Mongo Express web authentication in base64-encoded form.

### MongoDB Deployment (mongodb-deployment.yaml)
Runs a single MongoDB pod with credentials injected via environment variables from the secret. Exposes port 27017.

### MongoDB Service (mongodb-svc.yaml)
Exposes the MongoDB pod internally within the cluster on port 27017.

### ConfigMap (mongodb-configmap.yaml)
Contains the MongoDB service URL, used by Mongo Express for connecting to the database.

### Mongo Express Deployment (mongo-express-deployment.yaml)
Runs a single Mongo Express pod that connects to MongoDB using the secret and ConfigMap values for credentials and database URL. Exposes port 8081.

### Mongo Express Service (mongo-express-svc.yaml)
Exposes Mongo Express internally on port 8081; can be configured as LoadBalancer for external access.

### Ingress (mongo-ingress.yaml)
Routes HTTP traffic from mongodb-app.com to the Mongo Express service on port 8081, using Traefik as the ingress controller.