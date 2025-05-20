# How to see the HLD
- Open it at [diagram.net](https://app.diagrams.net/)

# Performance
- Peak RPS: 10000 requests/s
- Peak TPS: 600 transactions/s
- Number of predicted users: > 1 billion
- P99: <100ms 

# Explain the architecture
## Application runtime
- EKS as the container orchestration platform because it's suitable for this large microservices application
- EKS integrated with ALB controller for the Ingress layer
- Use AWS Application load balancer as the External LB of EKS cluster
- The ALB can handle million of request/second

- The FE application (client-side) will be deployed to S3 + Cloudfront
## Data layer
- AWS Managed Streaming for Kafka (MSK) is used as a message broker, many services communicate through Kafka
- Use RDS for Mysql with > 400GB storage to reach the max IOPS
- Elasticache for Redis is used for caching, and also queuing for notification services
## Secret management
- Use AWS Secret manager (SCM) to store secret of the Backend application
- Use External Secret Operator to fetch secrets from SCM to the EKS cluster
- Use AWS Systems Manager - Parameter store to store Frontend ENV
## CICD
- User AWS Code pipeline + CodeBuild to build the images
- Push Image to AWS ECR
- Use ArgoCD for CD
## Scaling
- Cluster Autoscaling: Karpenter will provide smart and effective scaling for cluster nodes
- Application scaling: KEDA
+ Scale based on Prometheus metrics 
- Scale strategy:
+ Scale API pods based on CPU/Memory and HTTP requests
+ Consumers will be scaled based on Kafka messages
## Observability
- Use Prometheus-stack to capture AWS, EKS cluster and application metrics
- Use Open telemetry for tracing
- Use Loki for logging