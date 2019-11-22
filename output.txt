apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  labels:
    app: hello
  name: broken-the-map
---
apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  labels:
    app: hello
  name: the-map
---
apiVersion: v1
data:
  .dockerconfigjson: ewogICJhdXRocyI6IHsKICAgICJyZWdpc3RyeS51dy5zeXN0ZW1zIjogewogICAgICAiYXV0aCI6ICJhc2RmYXNkZiIsCiAgICAgICJlbWFpbCI6ICJkZXZAbnVsbC5jb20iCiAgICB9CiAgfQp9
kind: Secret
metadata:
  name: docker-registry-key-264fk4cmmh
type: kubernetes.io/dockerconfigjson
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
  name: broken-the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello
    deployment: hello
  type: LoadBalancer
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
  name: the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello
    deployment: hello
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello
  name: broken-the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        deployment: hello
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: broken-the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: broken-the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: docker-registry-key
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello
  name: the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        deployment: hello
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: docker-registry-key-264fk4cmmh
