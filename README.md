# Ski Card Crawler Implementation Guide

This repository contains implementation guides for deploying the Ski Card Crawler application on either AWS or Kubernetes infrastructure.

## Quick Facts

| Aspect     | AWS                      | Kubernetes              |
|------------|--------------------------|------------------------|
| Setup Time | 4 days                   | 4 days                 |
| Complexity | Standalone infrastructure| Uses existing platform |
| Cost/Month | ~$65-70                  | Uses existing cluster + S3 |
| Cleanup    | Multiple AWS resources   | Single namespace      |

## Architecture Overview

### AWS Solution
The AWS implementation uses:
- EC2 t3.medium (Ubuntu 22.04)
- RDS PostgreSQL 11.22 (db.t3.micro)
- Application Load Balancer
- S3 Bucket
- CloudWatch monitoring

### Kubernetes Solution
The Kubernetes implementation uses:
- 2x Application Pods (500m CPU, 1Gi RAM)
- PostgreSQL StatefulSet (1Gi RAM, 10Gi Storage)
- Nginx Ingress
- S3 Bucket
- Prometheus/Grafana monitoring

## Technical Requirements
- Node.js v20
- PostgreSQL 11.22
- S3-compatible storage for screenshots
- HTTPS-enabled domain access
- GitHub repository access

## Implementation Timeline

### Day 1: Infrastructure Setup
- Network configuration
- Database provisioning
- Storage setup

### Day 2: Application Deployment
- Application server/container setup
- Environment configuration
- Initial deployment

### Day 3: Network & Security
- SSL/TLS configuration
- Domain setup
- Security hardening

### Day 4: Monitoring & Handover
- Monitoring setup
- Documentation
- Knowledge transfer

## Implementation Guides
Detailed implementation steps can be found in:
- [AWS Implementation Guide](aws.md)
- [Kubernetes Implementation Guide](k8s.md)

## Common Components

### Database
- PostgreSQL 11.22
- Basic CRUD operations
- No complex queries or functions
- Regular backup requirements

### Storage
- S3 bucket for screenshots
- Organized by resort/timestamp
- Accessed via pre-signed URLs
- No automatic deletion required

### Application
- Node.js v20 runtime
- Frontend + API server
- Basic authentication
- Screenshot capture capability

### Monitoring
- Health checks
- Resource utilization
- Error tracking
- Performance metrics

## Choice Guide

### Choose AWS if:
- No existing Kubernetes expertise
- Need standalone infrastructure
- Prefer managed services
- Want simpler initial setup

### Choose Kubernetes if:
- Have existing K8s cluster
- Need easier scaling
- Want standardized deployment
- Have DevOps expertise
