# Enterprise observability stack

This repository deploys the application and a lightweight observability stack into Minikube:

- NGINX Ingress with ModSecurity and OWASP Core Rule Set annotations for the application WAF
- Prometheus for Kubernetes pod metrics
- Grafana preconfigured with Prometheus as its default data source
- Fluent Bit as a node-level container log collector

## Minikube prerequisites

```sh
minikube start --cpus=4 --memory=8192
minikube addons enable ingress
kubectl apply -k environments/dev
```

The WAF annotations require the NGINX ingress controller. Add the ingress hostname locally:

```sh
echo "$(minikube ip) enterprise.local" | sudo tee -a /etc/hosts
```

Open the application at `http://enterprise.local`.

## Inspect the stack

Use port forwarding for the monitoring UIs:

```sh
kubectl -n observability port-forward svc/prometheus 9090:9090
kubectl -n observability port-forward svc/grafana 3000:3000
```

Fluent Bit enriches container logs with Kubernetes metadata and writes them to its pod output. Inspect collected application logs with:

```sh
kubectl -n observability logs -l app.kubernetes.io/name=fluent-bit --tail=100
```

The development overlay uses `emptyDir` storage for Minikube convenience, so Prometheus history and Grafana state are lost when their pods are recreated.
