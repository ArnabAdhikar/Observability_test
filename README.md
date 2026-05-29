# Observability Test

This project runs two Node.js services on Kubernetes and exposes metrics/tracing-friendly endpoints for observability testing.

- `service-a` listens on container port `3001`
- `service-b` listens on container port `3002`
- Kubernetes resources are managed from `kubernetes-manifest/`
- The app is deployed into the `dev` namespace

## Current Kubernetes Images

The deployments use public demo images:

```text
abhishekf5/demoservice-a:v
abhishekf5/demoservice-b:v
```

These are used so Kubernetes can pull the images without needing a private Docker registry login.

## Requirements

Install and start these before using the project:

- Docker Desktop
- Minikube
- kubectl

Check that Kubernetes is reachable:

```bash
kubectl get nodes
```

## Deploy The Application

Create the namespace if it does not already exist:

```bash
kubectl create namespace dev
```

If the namespace already exists, that command may print an error. That is okay.

Apply the manifests:

```bash
kubectl apply -k kubernetes-manifest/
```

Check the pods:

```bash
kubectl -n dev get pods
```

Expected result:

```text
service-a-deployment-...   1/1   Running
service-b-deployment-...   1/1   Running
```

Check the services:

```bash
kubectl -n dev get svc
```

Expected services:

```text
a-service
b-service
```

## Access From Browser

Because this setup uses Minikube on Windows/WSL2, the Minikube node IP may not work directly in a Windows browser. Use `kubectl port-forward` and open the app through `localhost`.

Start the port-forward:

```bash
kubectl -n dev port-forward service/a-service 8080:80
```

Keep that terminal open while using the app.

Open this in your browser:

```text
http://localhost:8080
```

Useful endpoints:

```text
http://localhost:8080/
http://localhost:8080/healthy
http://localhost:8080/metrics
http://localhost:8080/call-service-b
http://localhost:8080/logs
http://localhost:8080/example
```

Test the health endpoint from the terminal:

```bash
curl http://localhost:8080/healthy
```

## Run Traffic Test

After starting the port-forward, run:

```bash
./test.sh localhost:8080
```

This sends random requests to the app endpoints and helps generate metrics/logs.

## After The System Is Completely Switched Off

Use this checklist after a full shutdown, restart, or laptop power-off.

1. Start Docker Desktop.

2. Start Minikube:

```bash
minikube start
```

3. Confirm the cluster is running:

```bash
kubectl get nodes
```

4. Re-apply the Kubernetes manifests:

```bash
kubectl apply -k kubernetes-manifest/
```

5. Wait for the app pods:

```bash
kubectl -n dev get pods
```

Both pods should become `1/1 Running`.

6. Start browser access again:

```bash
kubectl -n dev port-forward service/a-service 8080:80
```

7. Open the app:

```text
http://localhost:8080
```

Important: `kubectl port-forward` is temporary. It stops when the terminal is closed, the machine shuts down, or the Kubernetes connection restarts. Run the port-forward command again whenever `localhost:8080` stops working.

## Common Troubleshooting

### Pods show `ImagePullBackOff`

Check the deployment image names:

```bash
kubectl -n dev describe pod <pod-name>
```

The current manifests should use:

```text
abhishekf5/demoservice-a:v
abhishekf5/demoservice-b:v
```

Apply the manifests again:

```bash
kubectl apply -k kubernetes-manifest/
```

### Browser times out on Minikube IP

If this URL times out:

```text
http://192.168.49.2:30002
```

Use localhost port-forward instead:

```bash
kubectl -n dev port-forward service/a-service 8080:80
```

Then open:

```text
http://localhost:8080
```

### Port 8080 is already in use

Use another local port:

```bash
kubectl -n dev port-forward service/a-service 8081:80
```

Then open:

```text
http://localhost:8081
```

### Clean default namespace

If old app resources were accidentally deployed into the `default` namespace, remove only those app resources:

```bash
kubectl -n default delete deployment service-a-deployment service-b-deployment
kubectl -n default delete service a-service b-service
```

Do not delete the built-in `kubernetes` service.

## Useful Commands

```bash
kubectl -n dev get all
kubectl -n dev get pods -o wide
kubectl -n dev logs deployment/service-a-deployment
kubectl -n dev logs deployment/service-b-deployment
kubectl -n dev rollout status deployment/service-a-deployment
kubectl -n dev rollout status deployment/service-b-deployment
```

