## Generate CPU Load and Test HPA in Kubernetes

### Verify Metrics Server

HPA requires metrics-server to collect CPU metrics.

Check if metrics server is running:
```bash
kubectl get deployment metrics-server -n kube-system
```

Verify metrics are available:
```bash
kubectl top nodes
```
If CPU metrics appear, the metrics server is working.

### Create Deployment

Create a file named deployment.yaml
```bash
vi deployment.yaml
```
Add the following configuration:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ng-deploy
  name: ng-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ng-pod
  template:
    metadata:
      labels:
        app: ng-pod
    spec:
      containers:
      - image: nginx
        name: nginx-ctr
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "1"
            memory: "500Mi"
```
Apply the deployment:
```bash
kubectl apply -f deployment.yaml
```

Verify pods:
```bash
kubectl get pods
```
### Create Horizontal Pod Autoscaler

Create HPA based on CPU utilization.
```bash
kubectl autoscale deployment ng-deploy --cpu 50% --min=2 --max=10
```

Check HPA status:
```bash
kubectl get hpa
```
**Get Pod Name**

Retrieve pod names.
```bash
kubectl get pods
```

Login to the container shell.
```bash
kubectl exec -it <pod name> -- /bin/bash
```
### Generate CPU Load

Run the following command:
```bash
yes > /dev/null &
```
Explanation:
* `yes` → continuously prints "y"
* `/dev/null` → discards the output
* `&` → runs the process in background

**Increase CPU Load**

Run multiple processes to increase CPU consumption.
```bash
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
```
Each command consumes *one CPU core.*

### Monitor CPU Usage

Open a new terminal and run:
```bash
kubectl top pod
```
This shows the CPU consumption increasing.

### Observe HPA Scaling

Monitor autoscaling:

```bash
kubectl get hpa -w
```
Also monitor pods:
```bash
kubectl get pods -w
```

### Stop the Load

Inside the pod, stop all yes processes.

```bash
kubectl exec -it <pod name> -- /bin/bash
```

```bash
killall yes
```
CPU usage will drop.

**Observe Scale Down**

Monitor HPA again:

```bash
kubectl get hpa -w
```
After CPU usage drops, Kubernetes automatically reduces pod replicas.

### Cleanup Resources

Delete HPA:

```bash
kubectl delete hpa ng-deploy
```
Delete deployment:

```bash
kubectl delete deployment ng-deploy
```






















