# Lab 0: Environment Setup

**Course:** Cloud Computing Security Essentials<br>
**Lab:** 0 Environment Setup<br>
**Date:** 29 July 2026

## Objective

Prepare and verify a local Linux-based environment for the cloud-security laboratory exercises. The environment requires container tooling, Kubernetes utilities, AWS command-line access, and cryptographic/authentication tools.

## Environment

The verification was performed from an Ubuntu terminal (WSL/Linux shell). The following tools were checked:

| Tool | Purpose | Verified version |
| :--- | :--- | :--- |
| Docker | Build and run containers | Docker 29.6.2, build dfc4efb |
| AWS CLI | Manage and interact with AWS services | aws-cli/2.36.9, Python/3.14.6, Linux/6.17.0-22-generic, exe/x86_64.ubuntu.24 |
| kind | Run local Kubernetes clusters using Docker | kind v0.23.0 |
| kubectl | Communicate with Kubernetes clusters | Client v1.36.3; Kustomize v5.8.1 |
| OpenSSL | Perform cryptographic and TLS-related operations | OpenSSL 3.0.13 (30 Jan 2024) |

## Verification Procedure and Evidence

### 1. Docker

The Docker installation was checked to ensure containerization tools are properly configured to run the lab environment:

```bash
docker --version
```

The command returned Docker version 29.6.2, confirming that the Docker command-line client is successfully installed.
<img width="584" height="62" alt="1" src="https://github.com/user-attachments/assets/134f4baf-006c-4562-b1de-82d2458a3843" />






### 2. AWS Command Line Interface

The AWS CLI installation was checked to verify that the system can communicate with and manage simulated cloud services:

```bash
aws --version
```

The output confirms AWS CLI version 2.36.10 is installed and running with Python 3.14.6 specifically within the WSL2 Ubuntu Linux environment.

<img width="670" height="64" alt="2" src="https://github.com/user-attachments/assets/6d47ba90-d97a-4165-adc0-816051b418fe" />

### 3. kind

The kind utility was checked to ensure the system is capable of spinning up local Kubernetes clusters for testing:

```bash
kind --version
```

The output confirms that kind version 0.23.0 is installed and ready to use.

<img width="584" height="62" alt="Screenshot 2026-08-05 042242" src="https://github.com/user-attachments/assets/c1c08c5f-476d-46d1-91ce-81d0bef45d04" />


### 4. kubectl

The kubectl tool was verified to ensure we can issue commands to control and manage the Kubernetes cluster once it is running:

```bash
kubectl version --client
```

The output confirms the kubectl client version v1.36.1 and Kustomize version v5.8.1 are successfully installed.

<img width="256" height="71" alt="4" src="https://github.com/user-attachments/assets/e0e483a9-68e5-4d25-9138-67a0e040411f" />



### 5. Cryptographic and OTP utilities

Finally, OpenSSL was verified to ensure the environment has the necessary cryptographic and certificate management tools required for later security labs:

```bash
openssl version
```

The output confirms OpenSSL version 3.5.5 (built on 27 Jan 2026) is installed and operational.

<img width="519" height="63" alt="5" src="https://github.com/user-attachments/assets/4660bd93-438c-4d79-89cd-07bc8babdedf" />


## Conclusion

The required command-line tools for the lab environment are installed and their versions have been recorded. Docker, AWS CLI, kind, kubectl and OpenSSL are available for subsequent cloud-computing security labs.
