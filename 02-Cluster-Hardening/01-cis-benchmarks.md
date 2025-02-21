# CIS Benchmarking for Security Hardening

## **Introduction**
This section provides an overview of **CIS (Center for Internet Security) Benchmarks** and their role in security hardening. These benchmarks offer best practices for securing operating systems, applications, and network devices to prevent unauthorized access and vulnerabilities.

---

## **Understanding CIS Benchmarks**
- CIS Benchmarks are a set of security guidelines for various technologies, including:
  - **Operating Systems** (Linux, Windows, macOS)
  - **Cloud Platforms** (AWS, Azure, Google Cloud)
  - **Networking Devices** (Cisco, Juniper, Palo Alto Networks)
  - **Software Applications** (Docker, Kubernetes, Nginx, Tomcat)
- These benchmarks provide step-by-step recommendations, commands, and remediation guidelines to secure systems effectively.

---

## **1. Securing Linux Systems with CIS Benchmarks**
- **Checking system security baselines:**
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install auditd
  sudo systemctl enable auditd --now
  ```
- **Key security risks include:**
  - **USB device attacks** (malware from physical access)
  - **Unauthorized user access** (admin logging in as root)
  - **Unrestricted service exposure** (firewall misconfigurations)
- **Best Practices:**
  - Disable USB ports if unnecessary:
    ```bash
    echo 'blacklist usb-storage' | sudo tee -a /etc/modprobe.d/blacklist.conf
    sudo update-initramfs -u
    ```
  - Restrict direct root access and enforce sudo:
    ```bash
    sudo passwd -l root
    sudo usermod -aG sudo username
    ```
  - Configure **firewall rules**:
    ```bash
    sudo ufw enable
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow ssh
    ```

---

## **2. System Configuration & Hardening**
- Steps to secure a system before deploying production applications:
  - Configure sudo properly:
    ```bash
    visudo  # Ensure only authorized users have sudo access
    ```
  - Enable **auditing and logging**:
    ```bash
    sudo auditctl -e 1
    sudo ausearch -m USER_LOGIN -ts today
    ```
  - Disable unnecessary services:
    ```bash
    sudo systemctl disable --now avahi-daemon
    sudo systemctl disable --now cups
    ```
  - Apply security patches:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

---

## **3. Automating Security Compliance with CIS Tools**
- **CIS CAT (CIS Configuration Assessment Tool):**
  - Automates security evaluations.
  - Compares system settings against CIS Benchmark recommendations.
  - Generates **HTML security reports** with pass/fail results.
- **How to use CIS CAT:**
  ```bash
  wget https://downloads.cisecurity.org/cis-cat-pro
  tar -xvf cis-cat-pro.tar.gz
  cd cis-cat-pro
  ./cis-cat.sh -benchmark Ubuntu_Linux
  ```

---

## **Conclusion**
CIS Benchmarks provide a structured approach to security hardening. Following these guidelines ensures that systems are protected against common threats while maintaining compliance with industry best practices. Implementing these benchmarks through hands-on command execution strengthens security posture and mitigates risk in production environments.

