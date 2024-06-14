### Deploy a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### Add Service to a pod
```yaml
# Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  # Labels are used to identify pods and used as selector fields
  labels:
    tut_name: Add-Service-To-Pods
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: add-service-to-pods
spec:
  # Selector Fields are use to filter pods using labels
  selector:
    tut_name: Add-Service-To-Pods
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

```