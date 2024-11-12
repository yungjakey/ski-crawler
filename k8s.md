# Ski Card Crawler - Kubernetes Implementation Guide

## Overview
Temporary implementation of ski card crawler using existing K8s cluster and AWS S3 for image storage.

## Prerequisites
- Access to existing K8s cluster with:
  - Ingress controller
  - Cert-manager
  - Monitoring stack
- AWS account access (for S3)
- Base domain for ingress
- Container registry access

## Architecture Components
- PostgreSQL on StatefulSet
- Node.js application deployment
- AWS S3 for screenshot storage
- Existing ingress for frontend access

## Timeline

### Day 1: Setup & Infrastructure
- [ ] Create namespace and basic resources
- [ ] Set up S3 bucket and access policies
- [ ] Configure database StatefulSet
- [ ] Verify storage and access patterns

### Day 2: Application Deployment
- [ ] Deploy crawler application
- [ ] Configure screenshot handling
- [ ] Set up ingress rules
- [ ] Initial connectivity testing

### Day 3: Testing & Adjustments
- [ ] Load testing
- [ ] Storage path verification
- [ ] Error handling validation
- [ ] Performance tuning if needed

### Day 4: Handover
- [ ] Documentation completion
- [ ] Stakeholder review
- [ ] Access handover
- [ ] Monitoring verification

## Implementation Details

### Storage Configuration
1. **S3 Bucket**
   - Region: eu-central-1
   - Lifecycle: No automatic deletion
   - Access: IAM role/credentials
   - CORS: Configured for frontend

2. **Database Storage**
   - PVC size: 10Gi
   - StorageClass: standard
   - Backup: Manual exports

3. **Screenshots**
   - Format: JPEG
   - Storage path: `<resort-id>/<timestamp>.jpg`
   - Retention: Maintain after cleanup

### Resource Requirements
```
Database:
- Memory: 1-2Gi
- CPU: 500m-1000m
- Storage: 10Gi

Application:
- Memory: 512Mi-1Gi
- CPU: 250m-500m
- Replicas: 2
```

### Network Access
- Ingress: HTTPS only
- Database: Internal access
- S3: Via IAM credentials

## Monitoring
- Use existing Prometheus/Grafana
- Basic metrics:
  - Crawler success rate
  - Response times
  - Storage usage
  - Database connections

## Backup Strategy
- Database dumps via CronJob
- S3 data remains
- Manifest backups

## Cleanup Procedure
1. Backup final database state
2. Verify S3 contents
3. Remove k8s resources:
   ```
   kubectl delete ns ski-crawler
   ```
4. Retain S3 bucket for potential future use

## Known Limitations
- No automatic scaling
- Basic monitoring only
- Manual database management
- Simple deployment strategy

## Security Notes
- S3 credentials in K8s secrets
- Database access internal only
- TLS via existing cert-manager
- Network policies recommended

## Operation Notes
- Manual scaling if needed
- Daily crawl via CronJob
- Log access via existing stack
- Screenshot URLs valid for 24h

## Troubleshooting
1. **Database Issues**
   - Check StatefulSet status
   - Verify PVC binding
   - Check resource usage

2. **Crawler Problems**
   - Check pod logs
   - Verify S3 access
   - Check memory usage

3. **Storage Issues**
   - Verify S3 credentials
   - Check bucket permissions
   - Monitor storage usage
