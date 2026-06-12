# afghan-dep-k8

Production-grade Kubernetes deployment configuration for a full-stack e-commerce application.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Kubernetes Cluster                       │
│                                                              │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐  │
│  │   Ingress   │     │   Ingress   │     │             │  │
│  │ shop.local  │     │api.shop.local│    │   Namespace │  │
│  └──────┬──────┘     └──────┬──────┘     │  afghan-dep │  │
│         │                   │            └─────────────┘  │
│         ▼                   ▼                              │
│  ┌─────────────┐     ┌─────────────┐                      │
│  │  Frontend   │     │   Backend   │     ┌─────────────┐  │
│  │  Service   │     │   Service   │     │   MongoDB   │  │
│  │  LB : 80    │     │  LB : 5000  │     │  StatefulSet│  │
│  └──────┬──────┘     └──────┬──────┘     └──────┬──────┘  │
│         │                   │                   │         │
│         ▼                   ▼                   ▼         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐  │
│  │  Frontend   │     │   Backend   │     │  MongoDB    │  │
│  │  Deployment │     │  Deployment │     │  Deployment │  │
│  │   (React)   │     │   (Node.js) │     │  (v7)       │  │
│  └─────────────┘     └──────┬──────┘     └─────────────┘  │
│                            │                              │
│                   ┌────────┴────────┐                     │
│                   │   HPA (1-10)    │                     │
│                   │   CPU: 80%      │                     │
│                   └────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

## Components

### Applications
| Component | Type | Port | Image |
|-----------|------|------|-------|
| Frontend | React SPA | 80 | `image` |
| Backend | Node.js API | 5000 | `image` |
| MongoDB | Database | 27017 | `mongo:7` |

### Kubernetes Resources (18 files)

#### Namespace
- `name-space.yaml` - `afghan-dep` namespace

#### Config & Secrets
- `config.yaml` - ConfigMap (client-url, backend-url)
- `secret.yaml` - Secrets (MongoDB, JWT, Email, Redis credentials)

#### Deployments
- `frontend-deployment.yaml` - React app with ConfigMap env var
- `backend-deployment.yaml` - Node.js API with Secrets + resource limits
- `mongodb-deployment.yaml` - StatefulSet with persistent storage

#### Services
- `frontend-svc.yaml` - LoadBalancer (port 80)
- `backend-svc.yaml` - LoadBalancer (port 5000)
- `mongo-svc.yaml` - ClusterIP (port 27017)

#### Ingress
- `frontend-ingress.yaml` - Routes `shop.local` → frontend-svc
- `backend-ingress.yaml` - Routes `api.shop.local` → backend-svc

#### Autoscaling
- `hpa.yaml` - Horizontal Pod Autoscaler (backend, 1-10 replicas, 80% CPU)
- `vpa.yaml` - Vertical Pod Autoscaler (MongoDB, resource recommendations)

#### Persistent Storage
- `upload-pv.yaml` / `uploads-pvc.yaml` - 5Gi for file uploads
- `mongo-pv.yaml` / `mongo-pvc.yaml` - 1Gi for MongoDB data

## Key Features

- **Auto-scaling**: HPA for backend (horizontal) + VPA for MongoDB (vertical)
- **Persistent Storage**: HostPath PVs with Retain policy
- **Secrets Management**: Kubernetes Secrets for sensitive data
- **Ingress Routing**: Host-based routing for frontend/backend separation
- **Resource Limits**: CPU/memory requests & limits on backend
- **StatefulSet**: MongoDB deployed as StatefulSet for stable storage

## Quick Deploy

```bash
# Create namespace first
kubectl apply -f k8s/name-space.yaml

# Apply all configs
kubectl apply -f k8s/

# Verify deployment
kubectl get all -n afghan-dep
```

## Environment Variables

### Backend
| Variable | Source |
|----------|--------|
| MONGODB_URI | secret |
| JWT_SECRET | secret |
| EMAIL_HOST, EMAIL_USER, EMAIL_PASS, EMAIL_FROM | secret |
| UPSTASH_REDIS_REST_URL, UPSTASH_REDIS_REST_TOKEN | secret |
| CLIENT_URL | configMap |

### Frontend
| Variable | Source |
|----------|--------|
| VITE_API_BASE | configMap |

## Tech Stack

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-%234ea94b.svg?style=for-the-badge&logo=mongodb&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-%23339933.svg?style=for-the-badge&logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-%2361dafb.svg?style=for-the-badge&logo=react&logoColor=black)