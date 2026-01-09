The Challenge
-----------------
When dealing with classified ads at scale, you're managing two types of data with very different characteristics, plus you need a robust way to deploy and run your application services:

Metadata: Small, structured information (ad title, description, price, seller ID, category, timestamps, status, location data)
--------
Images: Large binary files that users upload with their ads
--------
Application Services: Microservices that need to be containerized, deployed, and scaled reliably
-------
The Solution: Separation of Concerns Across Storage and Compute
------
DynamoDB for Metadata
Stores all the structured information about each ad in a highly optimized NoSQL database. Provides single-digit millisecond latency for queries like "find all active ads in this category" or "get ad details by ID". Supports complex access patterns through primary keys and secondary indexes (e.g., querying by user ID, category, date posted). Scales horizontally to handle millions of ad listings without performance degradation. The metadata typically includes an S3 reference (URL or key) pointing to where the actual images are stored.
---------
S3 for Images
Stores the actual image files (JPEGs, PNGs, etc.) which can be several megabytes each. Provides virtually unlimited storage capacity at a fraction of the cost of database storage. Offers built-in features like automatic redundancy, versioning, and lifecycle policies. Can integrate with CloudFront CDN for fast global image delivery. Images are referenced by unique keys/URLs stored in the DynamoDB metadata.
--------------
ECR (Elastic Container Registry) for Container Images
Acts as the private Docker registry for storing application container images. Each microservice (ad service, image processing service, search service, etc.) is packaged as a Docker image and pushed to ECR. Supports image versioning and tagging (e.g., ad-service:v1.2.3, ad-service:latest). Integrates seamlessly with EKS for automated deployments. Provides image scanning for vulnerabilities and encryption at rest. Enables CI/CD pipelines: build → test → push to ECR → deploy to EKS.
-----------
EKS (Elastic Kubernetes Service) for Application Deployment
Orchestrates and runs the containerized microservices pulled from ECR using Kubernetes. Manages pod specifications that define which container images to run, resource requirements (CPU/memory), environment variables, and IAM roles via IRSA (IAM Roles for Service Accounts). Deploys services across multiple availability zones for high availability using Kubernetes deployments and replica sets. Handles auto-scaling through Horizontal Pod Autoscaler (HPA) based on metrics like CPU usage, memory, or custom metrics. Performs health checks (liveness and readiness probes) and automatically replaces unhealthy pods. Integrates with Application Load Balancers (ALB) through the AWS Load Balancer Controller to distribute traffic across service instances. Provides zero-downtime deployments using rolling updates, blue/green deployments, or canary deployments.
---------
The Complete Workflow
Development & Deployment Pipeline
Developers write code for microservices (e.g., ad listing service, image upload service). Code is containerized using Docker. CI/CD pipeline builds the Docker image and runs tests. Successfully built images are tagged and pushed to ECR. Kubernetes manifests (deployments, services) are updated to reference the latest ECR image. EKS applies the updated manifests, performing a rolling deployment that gradually replaces old pods with new ones.
----------
Runtime Operations
User uploads an ad through the web/mobile interface. Request hits an Application Load Balancer. ALB routes to one of the Kubernetes pods running the ad service container in EKS. The ad service (running in EKS) uploads images directly to S3 (or uses pre-signed URLs for direct client upload). S3 returns URLs/keys for the uploaded images. The ad service saves metadata (including S3 references) to DynamoDB. When users browse ads, the search service (also running in EKS) queries DynamoDB for metadata. The application uses S3 URLs from metadata to display images (often served via CloudFront CDN).
-----------
Service Communication Example
Ad Service (EKS): Handles ad creation, updates, deletion - writes to DynamoDB and S3
Search Service (EKS): Handles ad queries and filtering - reads from DynamoDB
Image Processing Service (EKS): Generates thumbnails, applies watermarks - reads from and writes to S3
Notification Service (EKS): Sends alerts when ads are posted or receive offers
Why This Complete Architecture Works
Storage Layer Benefits
Cost efficiency: Storing binary data in databases is expensive; S3 costs pennies per GB. Performance: DynamoDB excels at structured queries; S3 excels at serving files. Scalability: Each storage system scales independently based on its workload.
---------
Deployment Layer Benefits
Consistency: ECR ensures all environments use the same vetted container images. Reliability: EKS handles container orchestration through Kubernetes, with self-healing capabilities and automatic pod recovery. Scalability: EKS can scale services up/down based on demand automatically using Kubernetes autoscaling mechanisms. Infrastructure as Code: Kubernetes manifests (deployments, services, ConfigMaps) are version-controlled and can be managed using tools like Helm or Kustomize. Security: Containers run with specific IAM roles via IRSA granting least-privilege access to DynamoDB and S3. Observability: EKS integrates with CloudWatch Container Insights, and supports Prometheus/Grafana for comprehensive logging and monitoring. Flexibility: Kubernetes provides advanced features like service mesh integration, custom resource definitions, and extensive ecosystem tools.
------
