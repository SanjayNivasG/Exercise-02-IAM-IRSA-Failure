# Exercise 02 – IAM / IRSA Failure Investigation

## Project Overview

This project simulates a production incident in Amazon EKS where an application suddenly lost access to Amazon DynamoDB because IAM Roles for Service Accounts (IRSA) was not configured correctly.

The objective was to investigate the issue, identify the root cause, implement the fix, and verify that the application could use the correct IAM role.

---

# Incident

## Problem Statement

The application was unable to read data from DynamoDB.

### Application Error

```text
botocore.exceptions.ClientError

An error occurred (AccessDeniedException)
when calling the GetItem operation:

User:
arn:aws:sts::123456789012:assumed-role/eks-nodegroup-role

is not authorized to perform:
dynamodb:GetItem
```

---

# Architecture

```
Application Pod
       │
       ▼
Kubernetes ServiceAccount
       │
       ▼
IAM Role (IRSA)
       │
       ▼
Amazon DynamoDB
```

---

# Root Cause Analysis

The Kubernetes ServiceAccount did not contain the required IRSA annotation.

Because IRSA was not configured, the application automatically used the EC2 Node IAM Role instead of the dedicated IAM Role assigned to the ServiceAccount.

Since the Node IAM Role did not have permission to perform `dynamodb:GetItem`, AWS returned an `AccessDeniedException`.

---

# Investigation Steps

* Reviewed the application error logs.
* Verified that the application Pod was running.
* Checked the ServiceAccount used by the Pod.
* Inspected the ServiceAccount configuration.
* Identified the missing IRSA annotation.
* Confirmed that the application was using the Node IAM Role.
* Updated the ServiceAccount with the correct IAM Role annotation.
* Restarted the Deployment.
* Verified that the configuration was corrected.

---

# Root Cause

* Missing IRSA annotation in the Kubernetes ServiceAccount.
* Application used the EC2 Node IAM Role.
* Node IAM Role did not have permission to access DynamoDB.

---

# Solution

Updated the ServiceAccount with the required IRSA annotation.

```yaml
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/payment-irsa-role
```

Applied the updated configuration and restarted the Deployment.

```bash
kubectl apply -f manifests/serviceaccount.yaml

kubectl rollout restart deployment payment -n payment
```

---

# Project Structure

```
Exercise-02-IAM-IRSA-Failure
│
├── README.md
│
├── manifests
│   ├── deployment.yaml
│   └── serviceaccount.yaml
│
└── evidence
    ├── application.log
    ├── investigation.txt
    ├── aws-sts-output.txt
    └── fixed-aws-sts-output.txt
```

---

# Commands Used

```bash
kubectl create namespace payment

kubectl apply -f manifests/serviceaccount.yaml

kubectl apply -f manifests/deployment.yaml

kubectl get pods -n payment

kubectl describe pod <pod-name> -n payment

kubectl get sa payment-sa -n payment -o yaml

kubectl apply -f manifests/serviceaccount.yaml

kubectl rollout restart deployment payment -n payment
```

---

# Key Learning Outcomes

* Understanding IAM Roles for Service Accounts (IRSA)
* Difference between Node IAM Role and Pod IAM Role
* Troubleshooting AWS permission issues in Amazon EKS
* Investigating AccessDeniedException errors
* Inspecting Kubernetes ServiceAccounts
* Applying IRSA annotations correctly
* Restarting Deployments after configuration changes
* Following a structured production incident investigation process

---

# Skills Demonstrated

* Kubernetes
* Amazon EKS
* IAM Roles for Service Accounts (IRSA)
* AWS IAM
* DynamoDB Access Control
* Kubernetes ServiceAccounts
* Production Incident Investigation
* Root Cause Analysis (RCA)
* YAML Configuration
* DevOps Troubleshooting

---

# Conclusion

This exercise demonstrates a realistic production troubleshooting scenario involving IAM Roles for Service Accounts in Amazon EKS. The investigation identified that the application was using the EC2 Node IAM Role due to a missing IRSA annotation. After updating the ServiceAccount and restarting the Deployment, the configuration was corrected, illustrating a structured approach to diagnosing and resolving AWS permission issues in Kubernetes.
