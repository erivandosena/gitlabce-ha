# GitLab Deployment on Kubernetes with High Availability

![GitLab CE](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e1/GitLab_logo.svg/1200px-GitLab_logo.svg.png)

## Introduction
This guide details the implementation of a Kubernetes stack for high availability of GitLab. It uses Helm for management, Longhorn as a distributed persistent storage solution, HAProxy as the Ingress controller, and includes monitoring support with Prometheus and Grafana.

### Key Features
- High Availability with storage replication.
- Ingress management with HAProxy and TLS support.
- Advanced monitoring of the cluster and GitLab.
- Simplified configuration and deployment with Helm.

---

## Project Structure

```plaintext
├── README.md                # Main documentation
├── values.yaml              # Helm Chart configuration for GitLab
├── longhorn                 # Longhorn storage configurations
│   ├── storageclass.yaml    # Longhorn StorageClass
│   ├── pvc.yaml             # Persistent Volume Claim
│   ├── deploy_longhorn.yaml # Longhorn installation
├── ingress                  # Ingress configuration for GitLab
│   ├── ingress_gitlab.yaml  # HAProxy configuration as Ingress
├── deployments              # Resources related to GitLab deployment
│   ├── namespace.yaml       # GitLab namespace
│   ├── helm_install.sh      # Deployment script with Helm
├── monitoring               # Prometheus and Grafana configurations
│   ├── prometheus.yaml      # Prometheus configuration
│   ├── grafana.yaml         # Grafana configuration
├── certificates             # TLS certificates for Ingress
│   ├── tls.crt              # TLS certificate
│   ├── tls.key              # Private key for the certificate
│   ├── ca.crt               # Root CA certificate
│   ├── trusted-ca.crt       # Additional trusted certificates
└── examples                 # Examples for testing and simulations
    ├── simulate_failures.md # Cluster failure simulation
    ├── test_gitlab.md       # Functional tests for GitLab
```

---

## Prerequisites

- Configured Kubernetes Cluster.
- Locally installed tools:
  - `kubectl`
  - `helm`
- HAProxy configured as the Ingress controller.
- Own TLS certificates (Wildcard) stored in the `certificates/` directory.

---

## Installation

### 1. Configure Longhorn
1. Apply the manifests to create the `StorageClass` and PVC:
   ```bash
   kubectl apply -f longhorn/storageclass.yaml
   kubectl apply -f longhorn/pvc.yaml
   kubectl apply -f longhorn/deploy_longhorn.yaml
   ```

### 2. Create TLS Secret
1. Create a Kubernetes Secret with the TLS certificates:
   ```bash
   kubectl create secret tls gitlab-tls \
     --cert=certificates/tls.crt \
     --key=certificates/tls.key \
     --namespace=gitlab
   ```

2. Create a ConfigMap with additional CA certificates:
   ```bash
   kubectl create configmap gitlab-ca-certificates \
     --from-file=certificates/ca.crt \
     --from-file=certificates/trusted-ca.crt \
     --namespace=gitlab
   ```

### 3. Deploy GitLab
1. Create the namespace for GitLab:
   ```bash
   kubectl apply -f deployments/namespace.yaml
   ```

2. Run the GitLab installation script with Helm:
   ```bash
   bash deployments/helm_install.sh
   ```

### 4. Configure Ingress with HAProxy
1. Apply the Ingress manifest:
   ```bash
   kubectl apply -f ingress/ingress_gitlab.yaml
   ```

---

## Monitoring

1. Configure Prometheus:
   ```bash
   kubectl apply -f monitoring/prometheus.yaml
   ```

2. Configure Grafana:
   ```bash
   kubectl apply -f monitoring/grafana.yaml
   ```

---

## Recovery in Case of GitLab Failure

If GitLab experiences a failure, follow these steps to recover and access repositories:

### 1. Check Pod Status
1. Use the command below to check the status of the pods:
   ```bash
   kubectl get pods -n gitlab
   ```
2. Identify the pods with errors and check their logs:
   ```bash
   kubectl logs <pod-name> -n gitlab
   ```

### 2. Restart Problematic Pods
1. Restart the pods with errors:
   ```bash
   kubectl delete pod <pod-name> -n gitlab
   ```

### 3. Recover Repository Access
1. Verify data persistence in the PVC:
   ```bash
   kubectl get pvc -n gitlab
   ```
2. If necessary, manually mount the PVC to check the data:
   ```bash
   kubectl exec -it <pod-name> -n gitlab -- ls /var/opt/gitlab/repo
   ```

### 4. Restore from Backup
1. List the available backups:
   ```bash
   ls /var/opt/gitlab/backups
   ```
2. Copy the backup to the pod and restore:
   ```bash
   kubectl cp <backup-file> <pod-name>:/var/opt/gitlab/backups -n gitlab
   gitlab-backup restore BACKUP=<backup-name>
   gitlab-ctl reconfigure
   ```
3. Automated process:
   https://server-gitlab.it.com/it/devops/gitlabce-backup-restore.git

---

## Testing and Simulations

- Refer to `examples/simulate_failures.md` for failure simulations.
- Refer to `examples/test_gitlab.md` for functional tests.