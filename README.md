# DevOps.NET-minimal-API
graph TD
    subgraph "Local Development PC"
        A[Developer] -->|Code Push| B[GitHub Repository]
    end

    subgraph "CI Pipeline (GitHub Actions)"
        B --> C{Trigger Workflow}
        C -->|Build .NET Image| D[Docker Hub]
        C -->|Update Image Tag| E[K8s Manifests /k8s]
    end

    subgraph "CD Pipeline (ArgoCD)"
        E -->|Sync Changes| F[ArgoCD Controller]
    end

    subgraph "Kubernetes Cluster (Vagrant VMs)"
        F -->|Deploy/Update| G[Master Node]
        G -->|Manage| H[Worker Node 1]
        G -->|Manage| I[Worker Node 2]
        
        subgraph "App Namespace"
            H --> J[.NET API Pod 1]
            I --> K[.NET API Pod 2]
            K <--> L[(MSSQL Server Pod)]
        end
        
        M[Ingress Controller] -->|Route: myapp.local| J
        M -->|Route: myapp.local| K
    end

    N[Browser] -->|Access App| M
