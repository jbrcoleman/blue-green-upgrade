# Blue-Green EKS Deployment

This repository demonstrates a blue-green deployment pattern for Amazon EKS clusters using Terraform, enabling seamless migration of stateless workloads between two Kubernetes clusters with minimal disruption.

## Overview

The blue-green deployment pattern allows platform teams to control infrastructure upgrades and migrations autonomously while providing a rollback mechanism and gradual traffic shifting capabilities.

### Architecture

- **Two EKS Clusters**: Blue (production) and Green (staging/new version)
- **Shared Infrastructure**: VPC, Route53 hosted zone, and supporting services
- **GitOps**: ArgoCD for continuous deployment
- **DNS Management**: External DNS with Route53 weighted routing
- **Traffic Control**: Gradual traffic shifting using Route53 weighted records

## Repository Structure

```
├── bootstrap/              # ArgoCD bootstrap configurations
│   ├── addons.yaml         # Cluster add-ons configuration
│   └── workloads.yaml      # Application workloads configuration
├── eks-blue/               # Blue cluster Terraform configuration
├── eks-green/              # Green cluster Terraform configuration
├── environment/            # Shared infrastructure (VPC, Route53, etc.)
├── modules/
│   └── eks_cluster/        # Reusable EKS cluster module
├── static/                 # Architecture diagrams and documentation
├── tear-down-applications.sh
├── tear-down.sh
└── terraform.tfvars.example
```

## Prerequisites

- AWS CLI configured with appropriate permissions
- Terraform >= 1.0
- kubectl
- An existing Route53 hosted zone
- GitHub repository for GitOps (ArgoCD)

## Quick Start

1. **Configure variables**:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars with your specific values
   ```

2. **Deploy shared infrastructure**:
   ```bash
   cd environment
   terraform init
   terraform plan
   terraform apply
   ```

3. **Deploy Blue cluster**:
   ```bash
   cd ../eks-blue
   terraform init
   terraform plan
   terraform apply
   ```

4. **Deploy Green cluster**:
   ```bash
   cd ../eks-green
   terraform init
   terraform plan
   terraform apply
   ```

## Configuration

### Key Variables

- `aws_region`: AWS region for deployment
- `environment_name`: Environment identifier
- `hosted_zone_name`: Existing Route53 hosted zone
- `eks_admin_role_name`: IAM role for EKS cluster admin access
- `gitops_addons_org`: GitHub organization for ArgoCD add-ons
- `gitops_workloads_org`: GitHub organization for application workloads

### Traffic Routing

Traffic distribution is controlled via Route53 weighted routing:

- **Blue cluster**: Initially receives 100% of traffic
- **Green cluster**: Gradually receives increased traffic during migration
- **Rollback**: Traffic can be quickly shifted back to Blue cluster

## Deployment Workflow

1. **Initial State**: Blue cluster serves 100% of traffic
2. **Green Deployment**: Deploy Green cluster with updates/upgrades
3. **Gradual Migration**: 
   - Shift 50% traffic to Green cluster
   - Monitor application performance
   - Gradually increase Green cluster traffic
4. **Complete Migration**: Route 100% traffic to Green cluster
5. **Cleanup**: Decommission Blue cluster after validation

## GitOps Integration

The solution uses ArgoCD for GitOps-based deployment:

- **Add-ons**: Managed via `bootstrap/addons.yaml`
- **Workloads**: Deployed through `bootstrap/workloads.yaml`
- **External DNS**: Automatically manages DNS records with cluster-specific ownership

## Cleanup

To tear down the infrastructure:

```bash
# Remove applications first
./tear-down-applications.sh

# Then destroy infrastructure
./tear-down.sh
```

## Security Considerations

- EKS clusters use IAM roles for service accounts (IRSA)
- Private subnets for worker nodes
- Security groups restrict access appropriately
- Secrets managed via AWS Secrets Manager

## Monitoring and Observability

The solution includes:
- AWS Load Balancer Controller for ingress
- External DNS for automatic DNS management
- ArgoCD for deployment visibility
- Route53 health checks for traffic routing decisions

## Based On

This implementation is based on the [AWS EKS Blueprints Blue-Green Upgrade Pattern](https://aws-ia.github.io/terraform-aws-eks-blueprints/patterns/blue-green-upgrade/).

