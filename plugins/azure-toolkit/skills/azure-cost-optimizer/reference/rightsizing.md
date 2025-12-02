# Rightsizing Guide

## VM Rightsizing
Monitor CPU, memory, and disk usage. Downsize if consistently under 40% utilization.

## Database Rightsizing
Review DTU or vCore utilization. Adjust tier based on actual usage patterns.

## Storage Optimization
- Use appropriate storage tiers (Hot, Cool, Archive)
- Enable lifecycle management
- Delete unused snapshots and backups

## Best Practices
- Monitor for 30+ days before changes
- Test in non-production first
- Use Azure Advisor recommendations
- Implement auto-scaling where possible
