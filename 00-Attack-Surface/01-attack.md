# Kubernetes Security: Understanding the Attack Surface

## Introduction
This document provides a structured and clear explanation of Kubernetes security vulnerabilities by analyzing a real-world attack scenario. The goal is to simplify complex security concepts, making them easy to understand while offering actionable security best practices to strengthen Kubernetes environments.

---

## **Attack Scenario: Exploiting a Kubernetes-Based Voting Application**

### **1. Initial Reconnaissance**
- A voting application consists of two key services:
  - `vote.com` (voting portal)
  - `result.com` (results display)
- These applications could be hosted on various platforms:
  - Cloud (AWS, GCP, Azure), on-premise, or PaaS solutions.
  - Running on containers, virtual machines, or Kubernetes clusters.
  
### **2. Identifying Infrastructure Weaknesses**
- The attacker sends network requests and notices both services respond with the same IP address, suggesting shared infrastructure.
- A port scan reveals that the host is running **Docker**, indicating containerized deployment.

### **3. Exploiting Insecure Docker Configuration**
- The attacker finds that the Docker API is exposed **without authentication**.
- Running `docker ps` with the host flag lists all running containers, exposing critical services.
- A privileged Ubuntu-based container is identified and used as an entry point.

### **4. Escalating Privileges Using Dirty COW**
- The attacker exploits the **Dirty COW** vulnerability, which allows escaping from the container to gain access to the underlying host.
- Missing utilities like `curl` and `wget` are installed to fetch an exploit script.
- Successfully executing the exploit grants root access to the host system.

### **5. Compromising the Kubernetes Cluster**
- The attacker discovers Kubernetes-related processes running on the host.
- A **Kubernetes Dashboard** is found exposed on port **30080**, **without authentication**.
- This grants visibility into the entire cluster, revealing namespaces, pods, and deployments.

### **6. Extracting Secrets and Modifying Data**
- A database pod (`DB pod`) is identified as storing vote data.
- The attacker extracts **database credentials** from environment variables.
- Using `psql`, they access the database and modify stored votes.
- A script is written and executed to manipulate election results.

### **7. Security Best Practices to Prevent This Attack**

✅ **Limit the Kubernetes Attack Surface:**
- Disable unauthenticated access to the **Docker API**.
- Remove public exposure of **Kubernetes Dashboard**.
- Implement **Role-Based Access Control (RBAC)** for access control.
- Ensure containers run as **non-root users** with restricted filesystem permissions.
- Regularly patch vulnerabilities like **Dirty COW**.
- Apply **network policies** to limit pod-to-pod communication.
- Enable **audit logging** to detect unauthorized access attempts.

---

## **Key Takeaways**
Understanding how attackers exploit misconfigurations is essential to securing Kubernetes environments. This scenario highlights common security pitfalls and reinforces best practices such as:
- **Hardening containers** using Seccomp, AppArmor, and SELinux.
- **Implementing strong RBAC policies** and securing `etcd`.
- **Encrypting Kubernetes Secrets** and restricting access.
- **Monitoring and detecting anomalies** with proper logging and alerting.

By addressing these vulnerabilities proactively, you can build a resilient and secure Kubernetes environment.

