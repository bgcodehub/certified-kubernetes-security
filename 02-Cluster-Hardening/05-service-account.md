# Kubernetes Service Accounts

## **Introduction**
Service accounts in Kubernetes are used by applications, controllers, and automated processes to interact with the cluster API. Unlike user accounts, which are managed externally, service accounts are native to Kubernetes and provide authentication and authorization for workloads.

---

## **1. User Accounts vs. Service Accounts**
- **User accounts** are for human users and are managed externally (e.g., LDAP, OAuth).
- **Service accounts** are for workloads (pods, controllers) that need access to the Kubernetes API.
- Every namespace in Kubernetes has a default service account named `default`.

```bash
# List all service accounts in a namespace
kubectl get serviceaccounts -n default
```

---

## **2. Creating a Service Account**
- Service accounts allow applications to authenticate with the Kubernetes API securely.

```bash
# Create a new service account
kubectl create serviceaccount my-service-account
```

- Verify the creation:
```bash
kubectl get serviceaccount my-service-account
```

---

## **3. Service Account Tokens and Authentication**
- When a service account is created, Kubernetes automatically generates a token for authentication.
- Tokens are stored as secrets and mounted into pods.

```bash
# List secrets associated with a service account
kubectl get secrets -n default
```

- Describe a secret to retrieve the token:
```bash
kubectl describe secret <secret-name>
```

---

## **4. Using a Service Account in a Pod**
- By default, Kubernetes automatically assigns the `default` service account to pods.
- To use a custom service account, specify it in the pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-service-account
  containers:
  - name: my-container
    image: my-app-image
```

```bash
# Deploy the pod
kubectl apply -f my-app-pod.yaml
```

---

## **5. Role-Based Access Control (RBAC) for Service Accounts**
- Service accounts need appropriate permissions to interact with the Kubernetes API.
- RBAC defines what a service account can do.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Apply RBAC roles and bindings
kubectl apply -f pod-reader-role.yaml
kubectl apply -f pod-reader-rolebinding.yaml
```

---

## **6. Managing Service Account Tokens (Post Kubernetes 1.22+ Updates)**
- Kubernetes v1.22 introduced improvements in service account token management.
- Tokens are now generated dynamically and must be explicitly requested.

```bash
# Generate a token for a service account
kubectl create token my-service-account
```

- Mounting tokens as projected volumes:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  serviceAccountName: my-service-account
  volumes:
  - name: token-vol
    projected:
      sources:
      - serviceAccountToken:
          path: token
          expirationSeconds: 3600
```

```bash
# Deploy the secure pod
kubectl apply -f secure-pod.yaml
```

---

## **Conclusion**
Service accounts provide a secure way for applications and workloads to authenticate with Kubernetes. By leveraging RBAC, managing service account tokens, and understanding how they interact with workloads, we can ensure better security and access control within the cluster.