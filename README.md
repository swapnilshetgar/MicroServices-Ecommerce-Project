# CloudCart E-Commerce Microservices

CloudCart is a production-style, full-stack e-commerce platform built with React, Node.js/Express, MySQL, Redis, Docker, and Kubernetes. It demonstrates service decomposition, API Gateway routing, persistent database storage, caching, ingress-based traffic management, metrics, and horizontal scaling.

[CloudCart architecture](CloudCart Kubernetes Architecture Infographic.png)

## Architecture

The complete request flow is:

```text
User
  |
  v
AWS Load Balancer
  |
  v
NGINX Ingress Controller
  |
  +-----------------------------+
  |                             |
  v                             v
Frontend (/), React             API Gateway (/api)
  |                             |
  +---------------------------->|
                                v
                     Backend microservices
                                |
                 +--------------+--------------+
                 v                             v
              MySQL                          Redis
           9 logical databases            Cart and cache
```

The Kubernetes cluster runs the frontend and API Gateway as independent workloads. The API Gateway routes requests to backend services, which communicate with MySQL or Redis according to their responsibility.

## Components

| Component | Responsibility |
|---|---|
| React Frontend | Product browsing, search, categories, cart, checkout, reviews, admin product creation |
| API Gateway | Central HTTP routing and Prometheus request metrics |
| Auth Service | Registration and login |
| User Service | User profile management |
| Product Service | Product catalog and cache-backed product reads |
| Inventory Service | Stock validation and updates |
| Cart Service | Shopping cart storage in Redis |
| Order Service | Order creation and order-item persistence |
| Payment Service | Mock payment success/failure processing |
| Shipping Service | Shipment and tracking creation |
| Notification Service | Order notifications |
| Review Service | Product reviews and ratings |
| MySQL StatefulSet | Persistent service data |
| Redis | Cart and product cache storage |
| NGINX Ingress | `/api/*` and frontend routing |
| Prometheus | Metrics collection and alert evaluation |
| Grafana | Monitoring dashboards and visualization |

## Technologies

- React and NGINX
- Node.js and Express.js
- MySQL 8
- Redis 7
- Docker and Docker Hub
- Kubernetes and kOps
- AWS Elastic Load Balancer
- NGINX Ingress Controller
- Helm
- ConfigMaps and Kubernetes Secrets
- PersistentVolumeClaims and EBS-backed storage
- Prometheus, ServiceMonitor, PrometheusRule, and Grafana
- Kubernetes probes and resource limits

## Repository Contents

The main setup document is [`microservices-project-new.txt`](microservices-project-new.txt). The [`create_cloudcart_fixed.sh`](create_cloudcart_fixed.sh) script generates the complete runnable project.

```text
create_cloudcart_fixed.sh
microservices-project-new.txt
architecture-new.png
architecture.png
```

Running the generator creates:

```text
cloudcart-ecommerce-microservices/
├── services/
│   ├── api-gateway/
│   ├── auth-service/
│   ├── user-service/
│   ├── product-service/
│   ├── inventory-service/
│   ├── cart-service/
│   ├── order-service/
│   ├── payment-service/
│   ├── shipping-service/
│   ├── notification-service/
│   └── review-service/
├── frontend/
├── database/
├── k8s/
└── scripts/
```

## Prerequisites

- AWS account with permissions for EC2, EBS, Route 53, IAM, and Elastic Load Balancing
- Kubernetes cluster created with kOps
- `kubectl`, `helm`, `docker`, and `git`
- Docker Hub account
- At least two worker nodes with sufficient CPU and memory
- NGINX Ingress Controller
- Metrics Server for resource metrics and optional HPA support

## Cluster Setup

Create or select the Kubernetes cluster:

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

Install Helm:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

Install the NGINX Ingress Controller:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace

kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

## Generate the Project

Run the generator from this directory:

```bash
chmod +x create_cloudcart_fixed.sh
./create_cloudcart_fixed.sh
cd cloudcart-ecommerce-microservices
```

You can override the generated defaults:

```bash
PROJECT=cloudcart-ecommerce-microservices \
DOCKER_USER=your-dockerhub-user \
IMAGE_TAG=v1 \
NAMESPACE=ecommerce \
MYSQL_ROOT_PASSWORD=root123 \
JWT_SECRET=cloudcart-jwt-secret \
./create_cloudcart_fixed.sh
```

Change the secrets from their default values before using the project outside a training environment.

## Build and Push Images

Log in to Docker Hub:

```bash
docker login
export DOCKER_USER=your-dockerhub-user
export IMAGE_TAG=v1
```

Build and push all frontend and backend images:

```bash
./scripts/build-push.sh
```

This publishes 12 images:

- `cloudcart-api-gateway:v1`
- `cloudcart-auth-service:v1`
- `cloudcart-user-service:v1`
- `cloudcart-product-service:v1`
- `cloudcart-inventory-service:v1`
- `cloudcart-cart-service:v1`
- `cloudcart-order-service:v1`
- `cloudcart-payment-service:v1`
- `cloudcart-shipping-service:v1`
- `cloudcart-notification-service:v1`
- `cloudcart-review-service:v1`
- `cloudcart-frontend:v1`

## Deploy to Kubernetes

Deploy the namespace, Secrets, ConfigMap, MySQL, Redis, services, frontend, and ingress:

```bash
./scripts/deploy.sh
```

Verify the deployment:

```bash
kubectl get all -n ecommerce
kubectl get pods -n ecommerce
kubectl get svc -n ecommerce
kubectl get ingress -n ecommerce
```

The API and frontend routes are exposed by the ingress:

| Path | Destination |
|---|---|
| `/api/*` | API Gateway |
| `/*` | React Frontend |

Access the application through the external IP or hostname from:

```bash
kubectl get svc -n ingress-nginx
kubectl get ingress -n ecommerce
```

Open the Ingress controller LoadBalancer address in a browser.

## Database and Storage

The MySQL initialization creates separate logical databases:

- `auth_db`
- `user_db`
- `product_db`
- `inventory_db`
- `order_db`
- `payment_db`
- `shipping_db`
- `notification_db`
- `review_db`

MySQL runs as a StatefulSet with a 5 GiB PersistentVolumeClaim. Redis stores cart data and cached product information.

For direct database access:

```bash
kubectl exec -it mysql-0 -n ecommerce -- mysql -uroot -p
```

Enter the `MYSQL_ROOT_PASSWORD` configured in `k8s/mysql-secret.yaml`.

## API Routes

| Route prefix | Service |
|---|---|
| `/api/auth` | Auth |
| `/api/users` | User |
| `/api/products` | Product |
| `/api/inventory` | Inventory |
| `/api/cart` | Cart |
| `/api/orders` | Order |
| `/api/payments` | Payment |
| `/api/shipping` | Shipping |
| `/api/notifications` | Notification |
| `/api/reviews` | Review |

All frontend API calls should use the `/api/...` paths so traffic remains on the same origin and passes through the API Gateway.

## Monitoring

Install the Prometheus community stack:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.service.type=LoadBalancer \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.ruleSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.serviceMonitorNamespaceSelector.any=true \
  --set prometheus.prometheusSpec.ruleNamespaceSelector.any=true
```

Label the application services for ServiceMonitor discovery:

```bash
kubectl label svc api-gateway auth-service user-service product-service \
  inventory-service cart-service order-service payment-service \
  shipping-service notification-service review-service \
  -n ecommerce cloudcart-monitor=true --overwrite
```

Apply the monitoring manifests:

```bash
kubectl apply -f k8s/monitoring-optional.yaml
kubectl get servicemonitor -n ecommerce
```

Useful Prometheus queries:

```promql
up{namespace="ecommerce"}
api_gateway_http_requests_total
product_service_http_requests_total
order_service_orders_created_total
payment_service_success_total
payment_service_failed_total
sum(rate(api_gateway_http_requests_total[2m]))
sum(rate(product_service_http_requests_total[2m]))
```

### Grafana

Get the Grafana LoadBalancer address:

```bash
kubectl get svc -n monitoring kube-prometheus-stack-grafana
```

Get the generated administrator password:

```bash
kubectl get secret kube-prometheus-stack-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
echo
```

The default username is `admin`.

## Verification and Troubleshooting

Run the generated verification script:

```bash
./scripts/verify.sh
```

Check pods and logs:

```bash
kubectl get pods -n ecommerce
kubectl describe pod <pod-name> -n ecommerce
kubectl logs -n ecommerce deploy/api-gateway --tail=50
kubectl logs -n ecommerce deploy/product-service --tail=50
```

Check Service endpoints and ingress:

```bash
kubectl get endpoints -n ecommerce
kubectl get ingress -n ecommerce
kubectl get svc -n ingress-nginx
```

Clear the product cache if product data is stale:

```bash
kubectl exec -it deploy/redis -n ecommerce -- redis-cli DEL products:list
```

Check MySQL:

```bash
kubectl exec -it mysql-0 -n ecommerce -- mysqladmin ping -uroot -p
kubectl logs -n ecommerce mysql-0 --tail=50
```

## Optional Resource Scaling

Install Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
kubectl top nodes
kubectl top pods -n ecommerce
```

If CPU-based HorizontalPodAutoscalers are required, create HPA resources targeting deployments such as `api-gateway`, `product-service`, or `order-service`, with a CPU utilization threshold of 60 percent.

## Cleanup

Delete only the application resources:

```bash
kubectl delete namespace ecommerce
```

Remove optional monitoring resources:

```bash
helm uninstall kube-prometheus-stack -n monitoring
helm uninstall ingress-nginx -n ingress-nginx
kubectl delete namespace monitoring ingress-nginx
```

Destroy the kOps cluster when the lab is no longer needed.

## Security Notes

- Replace default MySQL and JWT secrets.
- Do not commit real Docker credentials or Kubernetes secrets.
- Use stronger secrets from a secret manager for production.
- Enable TLS and a real DNS record for public deployment.
- Apply NetworkPolicies and authentication hardening before exposing the platform beyond a training environment.
