## 13. Change management and decommissioning (MAS 5.25-5.26)

MAS 5.25 governs changes to the AI, including training data, parameters and configuration. Infrastructure changes matter when they alter AI behavior or performance. The framework below classifies both.

**13.1 Change significance (5.25a)**

| Change | AI significance | Required action |
| --- | --- | --- |
| Model weights, quantization, prompt template, guardrail config | Significant for Tier 1-2 | Re-validation, re-approval before release |
| GPU host driver or guest driver version | Can change numerics and performance | Regression test in test domain. Re-baseline latency and output checks |
| vGPU profile or MIG layout | Changes capacity and isolation | Re-test. Update inventory and tier evidence |
| K8s version or GPU operator upgrade | Platform change | Run test suite. Staged rollout |
| ESXi patch without driver change | Low | Standard change. Compliance re-scan |
| NSX policy change on use-case groups | Security change | Review by security. Evidence updated |
| Reservation or placement rule change | Performance change | Capacity and latency check |
| Model Store tag or digest change | Significant | Blocked unless approved. Digest pinning |

Where a significant change occurs without prior review on a high-materiality use case, apply compensating controls such as enhanced monitoring (5.25a).

**13.2 Change control mechanisms (5.25b)**

- Everything as code: VKS cluster specs, NSX policy, VCF Automation templates, Avi config, model deployment manifests in Git. Pull request approval with human reviewer.
- Cluster desired state through vLCM images and cluster configuration profiles \[V on name in 9.x\]. Drift alerts in VCF Operations.
- Change records link to the inventory entry and to the evidence pack.
- Dynamic updates (5.25c): no mutable tags such as latest. No automatic cluster upgrades for Tier 2. Auto-retraining pipelines may run only within defined parameters and must pass the gate before promotion. Autoscaling of replicas is not a model update. Autoscaling that pulls a new image is.

**13.3 Rollback hierarchy**

Fastest and safest first.

1. Traffic rollback: Avi pool switch, blue/green or weighted. No state change.
2. GitOps revert or K8s rollout undo of the deployment.
3. Model rollback: re-point Model Runtime to the previous digest in Model Store.
4. Infrastructure rollback: vLCM image revert and remediation. Slower. Test it.
5. VM snapshot revert only for non-GPU VMs such as vector database nodes. Snapshots of vGPU VMs are reported unsupported \[V on vSphere 9\]. Keep audit logs off the VM. A revert erases them.
6. Restore from backup for data stores.

- Snapshots are short-lived rollback points, not backups. Do not leave them on Tier 2 databases.
- VKS nodes are replaced by the cluster controller. They are not snapshotted.

**13.4 Decommissioning (5.26)**

| Step | VCF mechanism | Evidence |
| --- | --- | --- |
| Identify dependencies | VCF Operations relationships. NSX flow data | Dependency report |
| Notify stakeholders | ITSM change record | Ticket |
| Disable and drain | Kill-switch L1-L3 | Run log |
| Revoke access | Remove IdP groups, service accounts, tokens, vault secrets | Revocation log |
| Remove network policy | Delete use-case NSX groups and DFW rules. Check no orphan allow rules | Policy diff |
| Remove workload | VCF Automation destroy. Delete namespaces, PVCs, VMs | Destroy log |
| Erase data and artifacts | Section 7.4 | Destruction record |
| Retain required records | Logs, validation pack, decision record under retention policy | Retention config |
| Update inventory | Set ai.lifecycle to retired. Close GRC record | Inventory history |
| Return GPU | Free vGPU or passthrough device for reallocation. Confirm no residual data per vendor guidance \[V\] | Capacity record |
