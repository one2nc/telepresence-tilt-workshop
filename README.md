# Welcome to Tilt and Telepresence Workshop

This demo has 4 dumb services to quickly demo tilt and telepresence on Kubernetes. 
## Services 
```mermaid
  graph TD;
     A[User] -->|For Signup and Login| B[User Service]
     A --> C{API Gateway}
     C -->|Fetch Product Info & Search Products By Name| P[Products Service]
     C -->|Create, Read and Update Orders| O[Orders]
     C -->|Process Payments| PS[Payment Service]
     PS -->|Update Order on Payment Complete| O
```