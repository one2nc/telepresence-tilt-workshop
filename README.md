# Welcome to the Tilt and Telepresence Workshop

This demo includes four microservices designed to demonstrate the usage of **Tilt** and **Telepresence** on Kubernetes.

## Services
```mermaid
graph TD;
   A[User] -->|Signup and Login| B[User Service]
   A --> C{API Gateway}
   C -->|Fetch Product Info & Search Products| P[Products Service]
   C -->|Create and Read Orders| O[Orders Service]
   C -->|Process Payments| PS[Payments Service]
   PS -->|Update Order on Payment Completion| O
```

### Service Overview

| Service           | Directory  | Purpose  | User Facing |
|------------------|------------|------------|-------------|
| **Users Service** | [telepresence-demo-user-service](./telepresence-demo-user-service/) | Handles user signup and login. Issues JWT tokens for authentication and authorization via the API Gateway. | ✅ |
| **Products Service** | [telepresence-demo-product-service](./telepresence-demo-product-service/) | Provides product catalog search functionality. | ❌ |
| **Orders Service** | [telepresence-demo-orders-service](./telepresence-demo-orders-service/) | Manages order creation, status updates (PENDING/PAID), and order retrieval. | ❌ |
| **Payments Service** | [telepresence-demo-payments-service](./telepresence-demo-payments-service/) | Facilitates payments and allows users to view past transactions. | ❌ |
| **API Gateway** | [telepresence-demo-api-gateway](./telepresence-demo-api-gateway/) | Protects the Orders and Payments API using JWT tokens (issued by the Users Service). This service allows users to create and view orders, make payments, and check payment status. | ✅ |

## User Flow
All API requests can be found in the Postman Collection: [Tilt-n-Telepresence-Demo.postman_collection.json](./Tilt-n-Telepresence-Demo.postman_collection.json).

### 1. User Signup & Authentication
- If not already registered, the user signs up via the **Users Service**.
- Upon signup, the user obtains a **JWT token**.

### 2. Product Search (No Authentication Required)
Users can browse products via the API Gateway:
```plaintext
GET /api/v1/products                   # View all products
GET /api/v1/products/category/:name    # Search by category
GET /api/v1/products/search/:name      # Search by name
GET /api/v1/products/:product_id       # Get details of a specific product
```

### 3. Creating an Order (JWT Token Required)
- API Endpoint: `/api/v1/order/create`
- Example cURL Request:
  ```bash
  curl --location 'http://api-gateway.api/api/v1/order/create' \
      --header 'Content-Type: application/json' \
      --header 'Authorization: Bearer <JWT Token>' \
      --data '{ "product_ids": ["L9ECAV7KIM", "OLJCESPC7Z"] }'
  ```
- Example Response:
  ```json
  {
      "order": {
          "id": "d7f4deda-d2f0-40a5-99a0-d0287a855edf",
          "user_id": "6cde8acb-c7e0-420e-8d10-7ada0a190430",
          "product_ids": ["L9ECAV7KIM", "OLJCESPC7Z"],
          "status": "pending"
      },
      "amount": 109.08
  }
  ```

### 4. Making a Payment
- API Endpoint: `/api/v1/payments/pay`
- Example cURL Request:
  ```bash
  curl --location 'http://api-gateway.api/api/v1/payments/pay' \
      --header 'Content-Type: application/json' \
      --header 'Authorization: Bearer <JWT Token>' \
      --data '{ "order_id": "67a06c81-c46c-43d5-a7dd-2f3a1c085132", "amount": 109.08 }'
  ```
- Example Response:
  ```json
  {
      "id": "7084ae0a-d386-4ecd-913d-17c72b408c2c",
      "user_id": "6cde8acb-c7e0-420e-8d10-7ada0a190430",
      "order_id": "67a06c81-c46c-43d5-a7dd-2f3a1c085132",
      "amount": 109.08,
      "status": "paid"
  }
  ```

### 5. Checking Payment Status
- API Endpoint: `/api/v1/payments/status/:payment_id`
- Example cURL Request:
  ```bash
  curl --location 'http://api-gateway.api/api/v1/payments/status/7084ae0a-d386-4ecd-913d-17c72b408c2c' \
      --header 'Authorization: Bearer <JWT Token>'
  ```
- Example Response:
  ```json
  {
      "id": "7084ae0a-d386-4ecd-913d-17c72b408c2c",
      "user_id": "6cde8acb-c7e0-420e-8d10-7ada0a190430",
      "order_id": "67a06c81-c46c-43d5-a7dd-2f3a1c085132",
      "amount": 109.08,
      "status": "paid"
  }
  ```




# Flow of the workshop
## Pre-requisites
1. A remote k8s cluster. (EKS, GKE, etc.)
2. Docker installed.
3. Linux/Mac preferred.
4. Install Tilt.
5. Install Telepresence.
6. Setup Postman.

## Flow
1. Develop the User Service using Tilt and run it inside the Kubernetes cluster.
2. Follow the same process for the Products and Orders services.
3. Deploy all three services to the remote Kubernetes cluster.
4. Set up Telepresence to connect to the remote cluster.
5. Develop the Payments Service locally using Telepresence to access dependent services inside the cluster.
6. Deploy the Payments Service to the cluster.
7. Repeat the same process as the Payments Service to develop and deploy the API Gateway Service using Telepresence.


# Developing Services with Tilt  

### 1. Navigate to the Service Directory  
We will use **Tilt** to develop the following services:  

| Service         | Directory |  
|----------------|--------------------------------------------|  
| Users Service  | [telepresence-demo-user-service](./telepresence-demo-user-service/) |  
| Products       | [telepresence-demo-product-service](./telepresence-demo-product-service/) |  
| Orders Service | [telepresence-demo-orders-service](./telepresence-demo-orders-service/) |  

### 2. Set Up a Python Virtual Environment  
Run the following commands to set up the development environment:  
```bash
python3 -m venv venv
source venv/bin/activate
make install-dev
```  

### 3. Start Developing Directly on the Cluster  
> **Important**: If developing on a remote cluster, Tilt may show the following error:  
>  
> _Stop! `name_of_k8s_context` might be production._  
> _If you're sure you want to deploy there, add:_  
> `allow_k8s_contexts("name_of_k8s_context")`  
> _to your Tiltfile. Otherwise, switch k8s contexts and restart Tilt._  

To resolve this, add the following line at the beginning of the `Tiltfile` in the service’s root directory:  
```python
allow_k8s_contexts("name_of_k8s_context")
```  

Then, start Tilt:  
```bash
tilt up
```  

### 4. Auto-Update on Code Changes  
Tilt will automatically update the running container inside the Kubernetes pod as you make changes.  

### 5. Shut Down Tilt  
To stop Tilt, press <kbd>Ctrl</kbd> + <kbd>C</kbd> and run:  
```bash
tilt down
```  

### 6. Deploy the Developed Version to the Cluster  
Once development is complete, deploy the service to the cluster:  
```bash
make deploy-dev
```  

# Developing a Service with Telepresence  

## Step 0: Install Telepresence and Connect to Your Cluster  

1. **Install the Traffic Manager on the Cluster**  
   ```bash
   telepresence helm install
   ```  
2. **Connect to the Cluster**  
   ```bash
   telepresence connect --namespace=api  
   ```  
   > **Note**: You can replace `api` with a different namespace if needed. For this demo, we will use `api` as our services are deployed there.  

## Step 1: Start Developing  

1. We will develop the following services using **Telepresence**. Navigate to the appropriate service directory:  

   | Service          | Directory |  
   |------------------|------------------------------------------------|  
   | Payments Service | [telepresence-demo-payments-service](./telepresence-demo-payments-service/) |  
   | API Gateway      | [telepresence-demo-api-gateway](./telepresence-demo-api-gateway/) |  

2. **Set Up a Python Virtual Environment**  
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   make install-dev
   ```  
3. You can now directly modify the codebase—no additional setup is required.  
4. **Run the Service Locally (Development Mode)**  
   ```bash
   cd ./src/
   uvicorn app:app --host 0.0.0.0 --port 8080 --reload  
   ```  

## Step 2: Deploy Changes to Kubernetes  

Once you're satisfied with your changes, deploy the updated service to the cluster:  
```bash
make deploy-dev
```  


# **Scenario: Telepresence Intercept Demo**  

## **Problem Statement**  
1. The **Payments Service** currently does not verify whether an order has already been paid, which may lead to **duplicate payments** for the same order.  
2. Setting up all dependent services **locally** is inconvenient.  
3. The cluster is accessible, and the Payments Service code is available.  

## **Solution**  

### **Step 1: Setup the Local Development Environment**  
1. Navigate to the **Payments Service** directory:  
   ```bash
   cd telepresence-demo-payments-service
   ```  
2. Set up a virtual environment (**if not done already**):  
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   make install-dev
   ```  
3. Run the **local instance** of the Payments Service:  
   ```bash
   cd src/
   uvicorn app:app --host 0.0.0.0 --port 8080 --reload  
   ```  

### **Step 2: Intercept the Service using Telepresence**  
1. Start a **Telepresence intercept** for `payments-svc`:  
   ```bash
   telepresence intercept payments-svc --port 8080  
   ```  
   > **How it works**:  
   > - Any requests from services inside the cluster (e.g., **API Gateway**) to `payments-svc` will be **re-routed** to your local instance instead of the one running in Kubernetes.  

### **Step 3: Modify the Payments Service**  
1. Open and modify [`telepresence-demo-payments-service/src/app.py`](./telepresence-demo-payments-service/src/app.py) to **prevent duplicate payments**:  

   ```python
   import os
   import uuid
   from typing import Dict

   import requests
   from fastapi import FastAPI, HTTPException

   from models import Payment

   app = FastAPI()

   # In-memory payment store
   payments: Dict[str, Payment] = {}

   ORDER_SERVICE_URL = os.getenv("ORDER_SERVICE_URL", "http://orders-svc.api")

   def check_if_order_already_paid_for(order_id: str):
       response = requests.get(f"{ORDER_SERVICE_URL}/orders/{order_id}")
       if response.status_code != 200:
           raise HTTPException(status_code=response.status_code, detail=response.json())

       order_info = response.json()
       order_status = order_info.get("status")

       if not order_status:
           raise HTTPException(status_code=500, detail="Internal Server Error")

       print(order_info)
       if order_status == "paid":
           raise HTTPException(status_code=403, detail="Already Paid")

   @app.get("/healthz", status_code=201)
   async def health():
       return {"message": "Hi Mom!"}

   @app.post("/payments", response_model=Payment)
   def process_payment(payment: Payment):
       payment.id = str(uuid.uuid4())
       payments[payment.id] = payment

       # Check if order is already paid
       check_if_order_already_paid_for(payment.order_id)

       # Update order status after successful payment
       update_order_url = f"{ORDER_SERVICE_URL}/orders/{payment.order_id}/status"
       response = requests.patch(update_order_url, params={"status": "paid"})

       if response.status_code != 200:
           raise HTTPException(status_code=500, detail="Internal Server Error")

       payment.status = "paid"
       return payment

   @app.get("/payments/{payment_id}", response_model=Payment)
   def get_payment(payment_id: str):
       if payment_id not in payments:
           raise HTTPException(status_code=404, detail="Payment not found")
       return payments[payment_id]
   ```

### **Step 4: Test the Changes**  
1. The **server will automatically reload** after code modifications.  
2. **Test the updated Payments Service** using the provided [Postman Collection](./Tilt-n-Telepresence-Demo.postman_collection.json):  
   - **Create an Order**  
   - **Make a payment** for the corresponding **Order ID**  
   - ✅ Payment should be **successful**  
   - **Attempt to pay for the same Order ID again**  
   - ❌ The request should **fail** with an error

### **Step 5: Deploy to Kubernetes**
1. Stop the intercept
   ```bash
   telepresence leave payments-svc
   ```
2. Deploy this version to Kubernetes.
   ```bash
   cd ..
   make deploy-dev
   ```

