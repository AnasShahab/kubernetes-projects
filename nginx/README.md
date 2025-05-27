# kubernetes-projects
## nginx
> This project demonstrates how to deploy a customized NGINX application on Kubernetes, complete with custom error pages, ingress routing via both Traefik and NGINX controllers, and a fallback mechanism for handling failures or direct IP access. It also includes a test pod for validating service connectivity.

### Breakdown
1. **nginx-configmap.yaml** provides content for the default page, 404 error page and custom nginx config that is used by **nginx-deployment.yaml**.
2. **nginx-deployment.yaml** creates 3 replica pods with nginx containers. The containers expose the app on port 80 by default.
3. **nginx-svc.yaml** creates a service for the nginx pods and exposes itself within the cluster on port 8080.
4. **nginx-ingress-traefik.yaml** creates an ingress resource that has rules to direct HTTP traffic incoming at host header *nginx-app.com* with path starting with */* to the nginx service. This routing will be done by a Traefik controller. The host *nginx-app.com* needs to resolve to the node IP exposed by the load balancer.
5. On *k3s* by default there is a Traefik controller that decides how to route HTTP/S requests. A *klipper-lb* service is also available that acts as a load balancer and assigns external IP.
6. Alternatively, **nginx-ingress-nginx.yaml** creates an ingresss resource to be used by a nginx controller. On *Minikube* ingress-nginx controller exists by default that runs as a NodePort or hostPort service, but not as a LoadBalancer. *Minikube* maps this port from the host (your machine) to the *Minikube* VM.

#### Fallback
1. By default *http://nginx-app.com* will show the index.html page. Requests for any non-existent page such as *http://nginx-app.com/hello* will display the 404.html page.
2. In case there are some faiures or when an IP is given instead of the FQDN a fallback mechanism is also provided.
3. **nginx-configmap.yaml** provides content for the fallback default page that is used by **nginx-deployment-fallback.yaml**.
4. **nginx-deployment-fallback.yaml** creates a pod with a nginx container that exposes itself on the default port 80.
5. **nginx-svc-fallback.yaml** creates a service for the nginx fallback pods and exposes itself within the cluster on port 8080.
6. **nginx-ingress-traefik-fallback.yaml** creates an ingress resource that has rules to direct HTTP traffic incoming at any host with parts starting with */*  to the nginx service. This is to be used by a Traefik controller. The host needs to be resolved to the node IP exposed by the load balancer.
7. Alternatively, **nginx-ingress-nginx-fallback.yaml** creates an ingresss resource to be used by a nginx controller.

### Overview of files:
#### configmap/nginx-configmap.yaml
> Creates a ConfigMap containing custom Nginx configuration and static HTML pages.
- index.html: Serves as the default page (*http://nginx-app.com*)
- 404.html: Provides custom content for 404 errors (*http://nginx-app.com/some_non_existent_page*)
- index-fallback.html: Used as a fallback page for special error handling scenarios, such as when accessing the service via the node’s IP instead of its hostname (*http://<IP_address_of_node>*)
- default.comf: Custom nginx config that:
    - Serves files from /usr/share/nginx/html.
    - Routes unmatched paths to the 404.html page.

#### deployment/nginx-deployment.yaml
> Deploys a replicated nginx application with 3 pods. It mounts custom HTML pages and an nginx configuration file from the *nginx-configmap.yaml* ConfigMap to serve personalized content and handle errors.

#### service/nginx-svc.yaml
> Defines a Kubernetes ClusterIP service that routes traffic on port 8080 to nginx pods on port 80. Can be switched to LoadBalancer type to expose the service externally.

#### pod/busybox.yaml
> Runs a BusyBox pod that fetches a page from the nginx service using a URL from the *nginx-configmap.yaml* ConfigMap (*nginx_url*) and then sleeps. Useful for testing service connectivity and environment variable injection.

#### ingress/nginx-ingress-traefik.yaml
> Defines an Ingress rule for routing HTTP traffic from nginx-app.com to the nginx-svc service on port 8080, using Traefik as the ingress controller. The machine from where traffic will originate needs to have the hostname *nginx-app.com* added as an entry to resolve against the node IP exposed by the load balancer. This can be done in */etc/hosts* or some other file responsible for local DNS lookup dpeending on the OS.

#### ingress/nginx-ingress-nginx.yaml
>Routes HTTP traffic from nginx-app.com (port 80) to the internal nginx service on port 8080 using the NGINX Ingress controller.

#### deployment/nginx-deployment-fallback.yaml
> Deploys a single NGINX pod using the nginx:alpine image. Serves a fallback HTML page from the *nginx-configmap* ConfigMap by mounting *index-fallback.html* to the default web path.

#### service/nginx-svc-fallback.yaml
> Defines a Kubernetes ClusterIP service that routes traffic on port 8080 to nginx fallback pod on port 80.

#### ingress/nginx-ingress-traefik-fallback.yaml
> Creates an Ingress rule for Traefik that routes all HTTP traffic on / to the fallback NGINX service on port 8080.

#### ingress/nginx-ingress-nginx-fallback.yaml
> Configures the NGINX Ingress controller to route all HTTP traffic on / to the fallback NGINX service (nginx-svc-fallback) on port 8080.
