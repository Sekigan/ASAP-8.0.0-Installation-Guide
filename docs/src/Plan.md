# ASAP Cloud Native Deployment Plan

## Pre-Installation Checklist

This document outlines the comprehensive plan for deploying ASAP (and Order Balancer) in a cloud-native environment. Use this checklist to ensure all prerequisites and steps are covered.

### Infrastructure & Environment

- [ ] Virtual Machine (VM) or Build Host  
  - Provision a VM or build host with Podman/Docker installed.  
  - Ensure adequate CPU, RAM, and disk space (see [[Prerequisites for Creating ASAP Image]] and [[Prerequisites for Creating an Order Balancer Image]]).  
  - Verify network connectivity to container registries and Kubernetes cluster.

- [ ] Container Registry  
  - Set up a private container registry (e.g., Harbor, GitLab Container Registry, AWS ECR, or OCI Container Registry) for storing ASAP and Order Balancer images.  
  - Configure authentication and access policies.

- [ ] Cartridge Definitions  
  - Review and finalize the list of cartridges to be used.  

- [ ] Version Control System (VCS)  
  - Choose a VCS (e.g., Git) and create repositories for:  
    - Dockerfiles and build scripts.  
    - Helm charts and Kubernetes manifests.  
    - Configuration files (properties, YAML).  
  - Set up CI/CD pipelines for automated builds and deployments.

- [ ] Kubernetes Cluster  
  - Provision a Kubernetes cluster (on-prem or managed like OKE, EKS, AKS, GKE).  
  - Ensure cluster version compatibility with ASAP/Order Balancer (see [[Planning and Validating Your Cloud Environment]]).  
  - Install Traefik

### Security & Access

- [ ] Secrets Management  
  - Use Kubernetes Secrets or external secret management (e.g., HashiCorp Vault, AWS Secrets Manager) for storing:  
    - Container registry credentials.  
    - Database passwords.  
    - API keys and tokens.  

- [ ] **TLS/SSL Certificates**  
  - Obtain certificates for ingress endpoints (See "[Setting Up ASAP Cloud Native for Incoming Access](https://docs.oracle.com/en/industries/communications/asap/8.0/cloud-native-deploy/integrating-asap.html#GUID-BB51B3EA-8683-469D-AA80-8DEE9A1B536F).").  
  - Ensure certificates are trusted and automatically renewed.

- [ ] **Authentication & Authorization**  
  - Define user roles and access controls for cluster and application.  

### Database & Storage

- [ ] **Database Provisioning**  
  - Provision a dedicated database instance or use managed DB service.  
  - Create schemas, users, and set up connection strings.  (See [[src/Planning you installation Order Balancer]] && "[Creating Oracle Database Tablespaces, Tablespace User, and Granting Permissions](https://docs.oracle.com/pls/topic/lookup?ctx=en/industries/communications/asap/8.0/cloud-native-deploy&id=ASPIG-GUID-F4F8FEAB-E65D-448E-850B-AF4193699A9A)")
  - Plan backup and recovery strategy (automated snapshots, point-in-time recovery).  
  - Test connectivity from the application.

### Application Images (ASAP & Order Balancer)

- [ ] **Build Images**  
  - Prepare Dockerfiles based on prerequisites.  
  - Build ASAP and Order Balancer images using Podman/Docker.  
  - Tag images with version numbers and push to the private registry.  
  - Verify image integrity and size.

- [ ] **Test Images Locally**  
  - Run containers locally to validate startup, configuration, and basic functionality.  
  - Check logs for errors.

### Deployment & Configuration

- [ ] **Helm Charts or Manifests**  
  - Prepare Helm charts or Kubernetes manifests for deploying ASAP and Order Balancer instances.  
  - Parameterize configurations (environment variables, secrets, resource limits).  
  - Include readiness/liveness probes, health checks.

- [ ] **Environment-Specific Configurations**  
  - Create configuration files for:  
    - Development.  
    - Staging/Testing.  
    - Production.  
  - Override settings (e.g., database URLs, logging levels, feature flags).

- [ ] **Install ASAP/Order Balancer Instances**  
  - Deploy to Kubernetes using Helm install/upgrade.  
  - Verify pods are running and services are exposed.  
  - Validate networking (ingress access, internal communication).

### Testing

- [ ] **Unit & Integration Tests**  
  - Run automated tests for ASAP and Order Balancer components.  
  - Test cartridge installation and functionality (see [[src/Working with Cartridges]]).

- [ ] **Performance/Load Testing**  
  - Simulate expected load to validate scalability and resource usage.  
  - Tune resource requests/limits.

- [ ] **User Acceptance Testing (UAT)**  
  - Test order submission, balance operations, and error handling.

- [ ] **Disaster Recovery Testing**  
  - Test restore from backups.  
  - Simulate failover scenarios (pod restarts, node failures).

### Monitoring, Logging & Alerting

- [ ] **Set Up Monitoring**  
  - Create dashboards (Grafana) for ASAP and Order Balancer performance.  
  - Define alerts for critical conditions (pod down, high latency, disk space).

- [ ] **Log Aggregation**  
   - Ensure logs are searchable and retained per policies.

### Documentation & Runbooks

- [ ] **Operational Documentation**  
  - Create runbooks for:  
    - Deployment steps.  
    - Upgrade/rollback procedures.  
    - Troubleshooting common issues.  
    - Backup and restore procedures.  
  - Update [[src/Prerequisites for Creating ASAP Image]] and [[src/Prerequisites for Creating an Order Balancer Image]] with actual values.

- [ ] Include troubleshooting tips.

### Staging Environment for Cartridge Testing

- [ ] **Create a Dedicated Testing Environment**  
  - Provision a separate Kubernetes namespace or cluster for testing new cartridges.  
  - Deploy a non-production ASAP/Order Balancer instance there.  
  - Test cartridge installation, upgrade, and removal without affecting production.

- [ ] **Automate Cartridge Validation**  
  - Create scripts to test cartridge functionality after installation.  
  - Document expected outputs and validation steps.

### Deployment Phases

+ [ ] **Phase 1: Build and Verify Images**  
   - Complete image builds and push to registry.  
   - Validate images in a sandbox environment.

+ [ ] **Phase 2: Staging Deployment**  
   - Deploy to staging cluster.  
   - Run all tests (unit, integration, load).  

+ [ ] **Phase 3: Production Deployment**  
   - Schedule maintenance window.  
   - Deploy to production cluster.  
   - Monitor closely during initial hours.

+ [ ]  **Phase 4: Handover**  
   - Train operations team.  
   - Deliver documentation.  
   - Establish ongoing support process.

### Post-Installation Tasks

- [ ] **Verify all services are healthy** (pods, endpoints).  
- [ ] **Validate backups are running**.  
- [ ] **Review alerting rules and dashboards**.  
- [ ] **Schedule periodic health checks** (weekly/monthly).  
- [ ] **Plan for updates** (image upgrades, cartridge updates, cluster upgrades).

---
## Next Steps

1. Review this plan with the team and adjust based on actual environment.  
2. Fill in specifics (registries, cluster names, database details).  
3. Start with the VM/build host setup and image building.

