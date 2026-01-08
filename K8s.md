KUBERNETES (K8s) COMPONENTS – ONE LINERS

CONTROL PLANE COMPONENTS

kube-apiserver
Entry point of Kubernetes; all requests and components communicate through the API Server

etcd
Distributed key-value store that holds the complete cluster state

kube-scheduler
Decides which node a pod should run on based on resources and constraints

kube-controller-manager
Runs controllers that continuously ensure the desired state of the cluster

cloud-controller-manager
Integrates Kubernetes with cloud provider services like load balancers and volumes

NODE (WORKER) COMPONENTS

kubelet
Agent running on each node that ensures containers are running as defined in Pod specs

kube-proxy
Handles networking and load-balancing for services across pods

Container Runtime
Software responsible for running containers (containerd, CRI-O, Docker)

CORE KUBERNETES OBJECTS

Pod
Smallest deployable unit in Kubernetes containing one or more containers

ReplicaSet
Ensures a specified number of pod replicas are always running

Deployment
Manages ReplicaSets and supports rolling updates and rollbacks

StatefulSet
Manages stateful applications with stable network identity and storage

DaemonSet
Ensures one pod runs on every node in the cluster

Job
Runs a task once and ensures successful completion

CronJob
Schedules jobs to run periodically

NETWORKING & ACCESS

Service
Provides a stable network endpoint to access a set of pods

ClusterIP
Exposes service internally within the cluster

NodePort
Exposes service on a static port on each node

LoadBalancer
Exposes service using a cloud provider load balancer

Ingress
Manages external HTTP/HTTPS access to services

Ingress Controller
Implements ingress rules (NGINX, ALB, Traefik)

CONFIGURATION & SECURITY

ConfigMap
Stores non-sensitive configuration data

Secret
Stores sensitive information like passwords and tokens

ServiceAccount
Provides identity for pods to interact with the Kubernetes API

RBAC
Controls access permissions within the cluster

SCALING & HEALTH

HPA (Horizontal Pod Autoscaler)
Automatically scales pods based on CPU or memory usage

VPA (Vertical Pod Autoscaler)
Adjusts CPU and memory resource requests for pods

Readiness Probe
Checks if a pod is ready to receive traffic

Liveness Probe
Checks if a pod is alive and restarts it if needed

Startup Probe
Ensures slow-starting containers are not killed prematurely

STORAGE

PersistentVolume (PV)
Represents a physical storage resource in the cluster

PersistentVolumeClaim (PVC)
Request for storage by a pod

StorageClass
Defines dynamic provisioning of storage volumes



KUBERNETES REQUEST FLOW (EXTERNAL → POD)

Client
↓
DNS (resolves domain to LB / Ingress IP)
↓
Cloud Load Balancer (ELB / ALB)
↓
Ingress Controller (NGINX / ALB / Traefik)
↓
Ingress Rules
↓
Service (ClusterIP)
↓
kube-proxy (iptables / IPVS)
↓
Pod
↓
Container (Application)

Response flows back the same path ↑

INTERNAL POD-TO-POD FLOW

Pod A
↓
Service DNS Name
↓
CoreDNS
↓
Service (ClusterIP)
↓
kube-proxy
↓
Pod B
↓
Container

KUBECTL / DEPLOYMENT FLOW

kubectl / CI-CD
↓
kube-apiserver
↓
AuthN / AuthZ (RBAC)
↓
Admission Controllers
↓
etcd (desired state stored)
↓
Controller Manager
↓
Scheduler
↓
kubelet
↓
Container Runtime
↓
Pod Running
