# Kubernetes Authentication

## **Introduction**
Authentication in Kubernetes is a crucial aspect of securing access to the cluster. Multiple types of users—including administrators, developers, and third-party applications—need secure access to interact with the Kubernetes API. This document covers authentication mechanisms used in Kubernetes and their implementation.

---

## **1. User Authentication in Kubernetes**
- Kubernetes does not store user accounts natively.
- Users are authenticated externally using:
  - **Static password files** (not recommended for production)
  - **Static token files**
  - **Client certificates**
  - **External authentication providers** like LDAP or Kerberos
  - **Service Accounts** for workloads

```yaml
# Example: Static token file format
abcdef1234567890,user1,uid123,"group1,group2"
```

---

## **2. Static Password & Token Authentication**
- **Static password files:** List usernames and passwords in a CSV file.
- **Static token files:** Store predefined tokens assigned to users.
- Authentication files must be referenced in the API server configuration.

```bash
# Example API server flag for static password authentication
--basic-auth-file=/etc/kubernetes/pwd.csv
```

```bash
# Example API server flag for token authentication
--token-auth-file=/etc/kubernetes/token.csv
```

**Security Concern:** Storing credentials in plaintext is highly insecure and should be avoided in production environments.

---

## **3. Client Certificate Authentication**
- Users authenticate with X.509 certificates signed by the Kubernetes Certificate Authority (CA).

### **Generating a Client Certificate**
```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=username/O=group"
openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user.crt -days 365
```

### **Using the Certificate with Kubectl**
```bash
kubectl config set-credentials username --client-certificate=user.crt --client-key=user.key
kubectl config set-context user-context --cluster=kubernetes --user=username
kubectl config use-context user-context
```

---

## **4. Integrating External Authentication Providers**
- Kubernetes can delegate authentication to external services:
  - **LDAP (Lightweight Directory Access Protocol)**
  - **Kerberos**
  - **OpenID Connect (OIDC)**
  
### **Example: Configuring OpenID Connect (OIDC)**
```yaml
--oidc-issuer-url=https://issuer.example.com
--oidc-client-id=kubernetes
--oidc-username-claim=email
--oidc-groups-claim=groups
```

---

## **5. Service Accounts for Workloads**
- Kubernetes manages service accounts internally for pods and controllers.
- Each pod can use a service account to interact with the API.

### **Creating a Service Account**
```bash
kubectl create serviceaccount my-service-account
```

### **Assigning a Role to the Service Account**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-role-binding
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
kubectl apply -f sa-role-binding.yaml
```

---

## **Conclusion**
Kubernetes authentication ensures that only authorized users and services can access cluster resources. By implementing client certificates, external authentication providers, and service accounts, security can be strengthened while minimizing reliance on insecure static credential files.

