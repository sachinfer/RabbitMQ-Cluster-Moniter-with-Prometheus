"# RabbitMQ-Cluster-Moniter-with-Prometheus" 
To monitor your RabbitMQ cluster with Prometheus and Grafana without using Helm, you can follow these steps:

Deploy Prometheus and Grafana:

Create a namespace for monitoring.
Deploy Prometheus.
Deploy Grafana.
Deploy RabbitMQ Exporter:

Deploy the RabbitMQ exporter to scrape metrics from RabbitMQ.
Configure Prometheus:

Update Prometheus configuration to scrape metrics from the RabbitMQ exporter.
Setup Grafana Dashboards:

Import Grafana dashboards to visualize RabbitMQ metrics.
Here are the steps with YAML configurations:

1. Create Namespace for Monitoring
Create a new namespace called monitoring:

yaml
Copy code
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
Apply the configuration:

sh
Copy code
kubectl apply -f namespace.yaml
2. Deploy Prometheus
Create a ConfigMap for Prometheus configuration:

yaml
Copy code
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: 'rabbitmq'
        static_configs:
          - targets: ['rabbitmq-exporter.monitoring.svc.cluster.local:9419']
Create a Deployment for Prometheus:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: prometheus-config-volume
        configMap:
          name: prometheus-config
Create a Service for Prometheus:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
  - protocol: TCP
    port: 9090
    targetPort: 9090
Apply the configurations:

sh
Copy code
kubectl apply -f prometheus-configmap.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
3. Deploy Grafana
Create a Deployment for Grafana:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana
        ports:
        - containerPort: 3000
Create a Service for Grafana:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
Apply the configurations:

sh
Copy code
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
4. Deploy RabbitMQ Exporter
Create a Deployment for RabbitMQ Exporter:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq-exporter
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rabbitmq-exporter
  template:
    metadata:
      labels:
        app: rabbitmq-exporter
    spec:
      containers:
      - name: rabbitmq-exporter
        image: kbudde/rabbitmq-exporter
        env:
        - name: RABBITMQ_URL
          value: http://<rabbitmq-username>:<rabbitmq-password>@rabbitmq.<your-namespace>.svc.cluster.local:15672
        ports:
        - containerPort: 9419
Create a Service for RabbitMQ Exporter:

yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq-exporter
  namespace: monitoring
spec:
  selector:
    app: rabbitmq-exporter
  ports:
  - protocol: TCP
    port: 9419
    targetPort: 9419
Apply the configurations:

sh
Copy code
kubectl apply -f rabbitmq-exporter-deployment.yaml
kubectl apply -f rabbitmq-exporter-service.yaml
5. Setup Grafana Dashboards
Access Grafana via the service created (you may need to set up port forwarding or use an ingress).

Import a RabbitMQ dashboard into Grafana:

Log in to Grafana.
Go to Configuration > Data Sources and add Prometheus as a data source.
Go to Create > Import and use a prebuilt RabbitMQ dashboard JSON file. You can find dashboard JSON files on the Grafana website or from the Grafana Dashboards repository.
This setup should allow you to monitor your RabbitMQ cluster using Prometheus and Grafana. Adjust the configurations and deployments according to your specific setup and requirements.
