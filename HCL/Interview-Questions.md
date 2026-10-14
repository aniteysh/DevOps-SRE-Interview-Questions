1. If a team is facing slow image download (pull) from Artifact Registry in a specific region, how would you identify the problem and improve performance across all regions?
**Solution:**
Immediate Diagnosis
 - Network & Routing: Check if traffic is routing over the public internet instead of private endpoints (like Private Service Connect/VPC endpoints), or if DNS is resolving to a distant regional endpoint.
 - Storage & Node Bottlenecks: Verify if disk IOPS or network bandwidth on the worker nodes are saturated during unpacking, or if the registry is hitting API rate limits/throttling.
- Image Footprint: Inspect image size and layer count to see if monolithic layers are causing the slowdown.
Architectural Fix
- Geo-Replication: Configure multi-region replication so worker nodes always pull locally within the same region over cloud backbone networks.
- Lazy Pulling: Implement lazy-pulling using formats like eStargz so containers start immediately while fetching remaining layers asynchronously.
- Node Caching & P2P: Optimize container runtime caching and deploy tools like Dragonfly for peer-to-peer layer sharing across large cluster nodes.
- Image Optimization: Enforce multi-stage builds and distroless base images to keep layer footprints minimal globally.”
2. After an OS update via MIG rolling update, a stateful database shows latency spikes and split-brain issues. How would you diagnose the root cause, mitigate immediately, and design a safer update process for future?
**Solution:**
Diagnose Root Cause
- Check Node Lifecycle & Quorum: A MIG rolling update destroys and replaces instances one by one. If it replaced quorum nodes simultaneously or too quickly, the database lost consensus (split-brain).
- Storage & Network Bottlenecks: Check if the new instances attached existing persistent disks correctly, or if IOPS throttling/network saturation caused replication lag.
- Kernel/OS Regressions: Look for kernel parameter changes (e.g., limits on file descriptors, swap settings, or TCP backlog) in the new OS image that hurt database performance.
Immediate Mitigation
- Restore Quorum: Isolate the unhealthy nodes, ensure the remaining healthy primary and replicas form a stable cluster, and fence off any rogue nodes to stop split-brain write conflicts.
- Rollback if Needed: Revert the MIG template to the previous golden image and roll back the instances if the OS update itself introduced a performance bug.
Safer Architecture for Future
- Custom Update Policies: Configure the MIG with strict max-surge and max-unavailable settings (e.g., max-unavailable: 0) to ensure zero downtime and strict one-at-a-node replacement.
- Stateful-Aware Orchestration: Move away from basic MIG rolling updates for stateful workloads. Use operator-managed databases (like StatefulSets with proper pod disruption budgets) that gracefully handle leader elections before a node is terminated.
- Canary Updates: Always test OS updates on a single non-production replica first to catch kernel or performance regressions before rolling out to the primary cluster.
3. What happens if two VPCs have overlapping IP (CIDR) ranges when trying to create a VPC peering connection? How can you fix this issue?
4. You are designing egress controls for multiple VPCs that need to access external APIs while restricting general internet access. Compare using centralized NAT gateways (via a transit gateway / hub) versus VPC endpoints (private service endpoints). Explain the trade-offs in terms of security, cost, and operations.
5. Design a multi-tenant VPC networking model for a SaaS platform that isolates tenant networks, allows optional connectivity between selected tenants, and supports centralized inspection for outbound traffic. Explain the network design, how you would prevent IP overlap, enforce tenant isolation, manage controlled tenant-to-tenant connectivity, and ensure, the solution scales efficiently with minimal opérational overhead.
6. A downstream service shows intermittent 500 errors, but traces indicate that requests are timing out before the error is returned. Explain how you would use tracing to identify whether the issue is in the downstream service, network, or client, and what other signals (metrics/logs) you would check to confirm the cause.
7. Design a scalable logging architecture using Cloud Logging for ingeśtion, BigQuery for analytics, and Cloud Storage for long-term archive. Explain how you would handle schema evolution, cost optimization, partitioning, and query performance at scale.
8. Your organization has hundreds of alert policies across many teams, leading to inconsistent escalation and unclear ownership. Design an operational model and governance plan to manage the alert lifecycle at scale, including onboarding, ownership, review, testing, and retirement.
9. How would you safely identify and remove unused service accounts without breaking running systems? Explain a simple approach using audit logs and monitoring.
10. A Kubernetes cluster uses a central secrets store with a CSI driver to mount database credentials into pods. After a migration, some pods intermittently receive credentials for the wrong tenants, causing data leaks. With minimal downtime constraints, explain the likely root causes, the immediate steps to stop the issue, and the long-term changes needed to prevent this type of failure in the future.
11. How would you ensure reproducible and auditable builds in Cloud Build across different environments?
12. A Cloud Build step is used to deploy an application to Kubernetes using Helm, but the deployment approach has security and reliability risks. What could be one such risk, and how would you make the deployment safer?
13. A CI pipeline running goloud deploy commands intermittently fails with 403 Forbidden, while the same command works locally using your user account. Describe how you would systematically diagnose identity, permissions, and environment differences, and design a secure fix that ensures consistent, non-interactive execution in CI without over-privileging access.
14. A Cloud Deploy pipeline performs hourly promotions to production, but outages have increased due to traffic growth. The pipeline uses canary releases without automated traffic control or rollback. Design a prioritized remediation plan to quickly reduce user impact and improve long-term deployment safety. Explain trade-offs and how you would measure success.
15. A support engineer discovers that failed messages are no longer visible in the primary subscription. What would you Investigate?
16. An application deployment fails even though the deployment template is valid. The error references organizational restrictions. What would you investigate? gcloud deployment-manager deployments create app-deployOutput:Request violates organization policy.
17. Review the following workflow extracted from a repository that intends to call a reusable workflow in another repo. Identify the issues that will prevent the call from succeeding, explain the impact of each, and propose corrected YAML.

```yaml
on:
  workflow_dispatch:

jobs:
  call:
    uses: org/repo/.github/workflows/template.yml@main
    with:
      env: production
    secrets:
      API_KEY: ${{ secrets.API_KEY }}
```

23. Design an approach using GitHub Actions to perform authenticated, auditable deployments to multiple cloud accounts (e.g., staging and production) while minimizing stored secrets and ensuring each deploy is tied to a specific Cl artifact and run. Explain how you enforce least privilege and how you correlate an incident back to the initiating workflow run.
24. Discuss the trade-offs and risk model of moving critical production deployment steps from GitHub Actions workflows running on hosted runners to a separate deployment service triggered by GitHub webhooks. Include security, reliability, auditability, and developer experience considerations.
25. How does the Deployment controller perform a rolling update?
26. A daily reporting workload creates thousands of short-lived Pods, causing API server and scheduler latency spikes. How would you redesign the cluster?
27. During a cluster upgrade, StatefulSet Pods restart successfully but application performance degrades significantly afterward. What would you investigate?
28. Terraform deployments are managed using Azure DevOps. Assume service connections and backend resources already exist. Write an Azure DevOps pipeline that Runs:
    1. terraform init
    2. terraform validate
    3. terraform plan
    4. Publishes the Terraform plan artifact.
    5. Requires approval before deployment to Production.
29. A frontend application cannot reach a backend Service using its DNS name after a deployment. What troubleshooting steps would you perform?
30. Create RBAC resources for a ServiceAccount named support-engineer.
    - Namespace: production-support
    - Allowed:
      - Get Pods
      - List Pods
      - Watch Pods
      - Read Pod logs
    - Not allowed:
      - Create Pods
      - Delete Pods
31. A service account token is suspected to be compromised. The application requires continued operation while the incident is investigated. What actions would you recommend?
32. Terraform fails when referencing resource attributes. What could be wrong?
33. Discuss the consequences and recovery options if a Terraform apply failed halfway through due to provider exception, leaving some resources created and state not updated. How do you safely reconcile state and infrastructure?
34. What is the benefit of using private Terraform modules?
35. A team wants to enforce policy checks before Terraform apply. How can CI/D help?
36. A Sentinel policy blocks a remote run due to restricted instance typeś.resource "aws_instance" "app" { instance_typeHow should you handle excéptions without compromising governance?
37. A production application unexpectedly reverted to an older configuration version. Which GitOps components would you examine?
38. For multi-cluster GitOps, what are the complexities of detecting drift correctly and consistently across clusters, and how would you design a system to minimize false positives while still catching real divergence quickly?
39. You need to meet a compliance requirement that all secrets must be encrypted with a customer-managed key (CMK) stored in HSM in a separate subscription. Explain how to implement this for Key Vault secrets, including cross-subscription considerations; limitations, and how to validate that encryption-at-rest uses the correct CMK.
40. In a high-traffic microservices system, how would you design trace sampling and data retention to balance cost and useful insights in Splunk Observability?
41. Design an approach for validating that a set of business KPIs derived from multiple sources (metrics, events, logs) remains consistent after changes in instrumentation. Include monitoring, testing, and alerting mechanisms you would put in place.
42. You need to parse a simple CSV file where fields may contain commas inside quoted fields. Why is using naive 'IFS-, read loops unsafe, and what are two practical approaches to handle this in shell-based automation?
43. A high-traffic API suddenly has latency spikes and error rates increase after a configuration change. Logs show a flood of identical WARN messages from one library across many hosts, causing disks to saturate and triggering alerts. You cannot take downtime; disk space is limited and rotation is active. Using tail/less/journalctl/grep/awk, outline a prioritized incident, response plan to stop the immediate impact, preserve diagnostic, data, find the underlying change, and recommend long-term fixes. Discuss trade-offs and risks for each step.