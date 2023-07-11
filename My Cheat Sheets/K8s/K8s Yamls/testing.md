```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
       - name: webapp
         image: darktang3nt/webapp:debug
         imagePullPolicy: Always
         ports:
          - containerPort: 5000
      nodeSelector:
        kubernetes.io/arch: amd64
---
apiVersion: v1
kind: Service
metadata:
  name: pts-service
  annotations:
    traefik.ingress.kubernetes.io/service.sticky.cookie: "true"
    traefik.ingress.kubernetes.io/service.sticky.cookie.name: pts
spec:
  # ingressClassName: traefik
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pts-ingress
  annotations:
    spec.ingressClassName: traefik
    traefik.backend.loadbalancer.stickiness: "true"
    traefik.backend.loadbalancer.stickiness.cookieName: "ptsss"
spec:
  rules:
  - host: "pts.darktang3nt.cloud"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: pts-service
            port:
              number: 80
```