# Kubernetes Multi-Namespace Microservices Deployment (Bookinfo)

This repository demonstrates how to deploy the [Istio Bookinfo Application](https://istio.io/latest/docs/examples/bookinfo/) across multiple, isolated Kubernetes namespaces. It specifically addresses the challenges of **cross-namespace service discovery** and how to resolve them using environment variables and Fully Qualified Domain Names (FQDN).

## 🏗️ Application Architecture

The Bookinfo application is broken into four separate microservices:
- **Productpage (Python):** The entry point that calls the details and reviews services.
- **Details (Ruby):** Contains book information.
- **Reviews (Java):** Contains book reviews and calls the ratings service (Versions v1, v2, and v3).
- **Ratings (Node.js):** Contains book ranking information.

![Application Architecture](https://istio.io/latest/docs/examples/bookinfo/noistio.svg)

## 📁 Project Structure

```bash
bookinfo-different-namespace/
├── README.md
└── apps/
    ├── details.yaml     # Deployed to 'details' namespace
    ├── productpage.yaml   # Deployed to 'productpage' namespace
    ├── ratings.yaml     # Deployed to 'ratings' namespace
    └── reviews.yaml     # Deployed to 'reviews' namespace
```

---

## ❌ The Problem: Cross-Namespace Communication Failure

By default, Kubernetes services use a "short name" DNS resolution. If a pod in the `productpage` namespace tries to reach a service named `reviews`, Kubernetes looks for it **only within the `productpage` namespace**.

When we split the microservices into separate namespaces (`productpage`, `details`, `reviews`, `ratings`), the application fails because:
1. The **Productpage** cannot find `details` or `reviews`.
2. The **Reviews** service cannot find `ratings`.

**Result:** The web UI will load but show errors like *"Error fetching product details"* and *"Review service is currently unavailable"*.

---

## ✅ The Solution: FQDN & Environment Variables

### 1. Fully Qualified Domain Names (FQDN)
To communicate across namespaces, we must use the service's FQDN following this format:
`[service-name].[namespace].svc.cluster.local`

**New internal addresses:**
- `details.details.svc.cluster.local`
- `reviews.reviews.svc.cluster.local`
- `ratings.ratings.svc.cluster.local`

### 2. Identifying Override Variables via Source Code
The Bookinfo microservices are programmed to accept custom hostnames via environment variables.

#### **In Productpage (Python)**
Checking [productpage.py](https://github.com/istio/istio/blob/master/samples/bookinfo/src/productpage/productpage.py):
The code looks for `DETAILS_HOSTNAME` and `REVIEWS_HOSTNAME`. If not found, it defaults to the short names.
```python
detailsHostname = os.environ.get('DETAILS_HOSTNAME', 'details')
reviewsHostname = os.environ.get('REVIEWS_HOSTNAME', 'reviews')
```

#### **In Reviews (Java)**
Checking [LibertyRestEndpoint.java](https://github.com/istio/istio/blob/master/samples/bookinfo/src/reviews/reviews-application/src/main/java/application/rest/LibertyRestEndpoint.java):
The code looks for `RATINGS_HOSTNAME`.
```java
private final static String ratings_hostname = System.getenv("RATINGS_HOSTNAME") == null ? "ratings" : System.getenv("RATINGS_HOSTNAME");
```

---

## 🚀 Deployment Steps

### 1. Setup Infrastructure
```bash
# Create Cluster (using Kind)
./setupkindcluster134-audc-cluster.sh

# Create Namespaces
kubectl create ns productpage
kubectl create ns details
kubectl create ns reviews
kubectl create ns ratings
```

### 2. Update and Apply Manifests
Inject the required FQDNs into the Deployment manifests.

**For `apps/productpage.yaml`:**
```yaml
env:
  - name: DETAILS_HOSTNAME
    value: "details.details.svc.cluster.local"
  - name: REVIEWS_HOSTNAME
    value: "reviews.reviews.svc.cluster.local"
```

**For `apps/reviews.yaml` (Apply to v1, v2, and v3):**
```yaml
env:
  - name: RATINGS_HOSTNAME
    value: "ratings.ratings.svc.cluster.local"
```

**Apply all manifests:**
```bash
kubectl apply -f apps/details.yaml -n details
kubectl apply -f apps/ratings.yaml -n ratings
kubectl apply -f apps/reviews.yaml -n reviews
kubectl apply -f apps/productpage.yaml -n productpage
```

### 3. Verify Access
Since the cluster runs in Docker (on macOS), use port-forwarding to access the Productpage:

```bash
kubectl port-forward svc/productpage 9080:9080 -n productpage
```

Open `http://localhost:9080/productpage` in your browser. You should now see the complete page with details and reviews loading correctly from their respective namespaces.

---

## 📝 Summary Table

| Service | Target Service | Namespace | Environment Variable | Value |
| :--- | :--- | :--- | :--- | :--- |
| **Productpage** | Details | `details` | `DETAILS_HOSTNAME` | `details.details.svc.cluster.local` |
| **Productpage** | Reviews | `reviews` | `REVIEWS_HOSTNAME` | `reviews.reviews.svc.cluster.local` |
| **Reviews** | Ratings | `ratings` | `RATINGS_HOSTNAME` | `ratings.ratings.svc.cluster.local` |