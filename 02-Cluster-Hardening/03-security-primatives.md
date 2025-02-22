# Kubernetes Security Primitives

## **Introduction**
Kubernetes is the leading platform for container orchestration, making security a critical concern for production-grade deployments. This document provides an overview of essential security primitives in Kubernetes, focusing on host security, API server access control, authentication, authorization, and network security.

---

## **1. Securing Kubernetes Hosts**
- All access to Kubernetes hosts must be restricted.
- **Best practices:**
  - Disable **root access**.
  - Disable **password-based authentication**, enforcing SSH **key-based authentication only**.
  - Secure both **physical and virtual infrastructure** hosting Kubernetes.
  
```bash
# Disable root login
sudo passwd -l root

# Enforce SSH key-based authentication
sudo sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## **2. Controlling Access to the Kubernetes API Server**
- The API server is the core of Kubernetes, enabling interaction via `kubectl` or direct API calls.
- **Two critical security decisions:**
  1. **Who can access the API server?** (Authentication)
  2. **What actions can they perform?** (Authorization)

### **Authentication Methods:**
- **User-based authentication:**
  - Usernames/passwords stored in static files (not recommended for production).
  - Token-based authentication.
  - Client certificates.
  - Integration with external authentication providers like LDAP.
- **Machine authentication:**
  - Service accounts for pods and controllers.
  
```bash
# Generate a certificate for a user
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=username/O=group"
openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user.crt -days 365
```

---

## **3. Authorization Mechanisms**
- Once authenticated, actions are governed by authorization controls.
- Kubernetes supports multiple authorization modules:
  - **RBAC (Role-Based Access Control)** - Defines user/group permissions.
  - **ABAC (Attribute-Based Access Control)** - Legacy, not recommended.
  - **Webhook authorization** - External policy evaluation.

```bash
# Example: Creating an RBAC role to allow listing pods
kubectl create role pod-reader --verb=get,list --resource=pods
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=username
```

---

## **4. Securing Cluster Communication**
- All Kubernetes component communication must be encrypted using **TLS**.
- Kubernetes secures:
  - etcd (database storing cluster state)
  - API Server, Controller Manager, Scheduler, and Worker Nodes.
  
```bash
# Verify TLS is enabled for etcd
kubectl get pods -n kube-system -o wide | grep etcd
```

---

## **5. Implementing Network Policies**
- By default, pods in Kubernetes can communicate with each other freely.
- Network policies help **restrict communication between pods**.
- Policies are enforced using network plugins such as **Calico, Cilium, or Weave**.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Ingress
  - Egress
```
```bash
# Apply the network policy
kubectl apply -f deny-all.yaml
```

---

## **Conclusion**
Understanding and implementing security primitives in Kubernetes ensures a robust and secure cluster. By securing host access, controlling API server authentication, enforcing RBAC, encrypting communication, and applying network policies, organizations can effectively mitigate security risks in their Kubernetes environments.

