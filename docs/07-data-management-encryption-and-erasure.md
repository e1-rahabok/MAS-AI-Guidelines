## 7. Data management, encryption and erasure (MAS 5.4)

VCF controls clause 5.4(d), (e) and part of (f). Clauses 5.4(a), (b), (c) and (g) need data-governance tooling.

**7.1 Clause 5.4(a)-(g) mapping**

| Clause | VCF control | Configuration | Evidence | Gap |
| --- | --- | --- | --- | --- |
| (a) Fit for purpose | None | Data catalog and use-case review | Review record | Out of VCF scope |
| (b) Representativeness | None | Test data sets in the test domain | Evaluation report | Out of VCF scope |
| (c) Quality, drift | Compute and storage for quality pipelines. PAIS data indexing and retrieval connectors | Pipelines on VKS | Pipeline run logs | Quality metrics need a data-quality tool |
| (d) Classification | Tag ai.data-class drives storage policy, NSX group and key provider | One vSAN storage policy per class. Datastore or share per class | Policy compliance report | Classification itself is a data-owner act |
| (e) Security | Encryption at rest and in transit. Key management. Destruction | Sections 7.2-7.4 | Encryption status export, KMS audit log | Defined by FI data destruction standard |
| (f) Privacy | Residency and access logging | ai.jurisdiction tag with DRS VM-Host rules and no cross-region replication. vCenter and NSX audit logs | Placement report, access logs | Consent management is out of scope |
| (g) Lineage | Artifact provenance in Model Store. Pipeline code in Git | Harbor OCI metadata, signed images | Registry audit log | Data lineage needs a catalog tool |

**7.2 Encryption layers**

| Layer | Control | Scope | Protects against | Does not protect against |
| --- | --- | --- | --- | --- |
| vSAN data-at-rest | AES-256, ESA encrypts after compression ([VMware](https://www.vmware.com/docs/vmw-vsan-encryption-services)) | Cluster | Removed or stolen devices | Authorized vSphere or guest access |
| vSAN data-in-transit | Cluster-level. Remote datastore support added in VCF 9.1 | Cluster and mounts | Network sniffing between hosts | Compromised guest |
| VM Encryption | Storage policy per VM, key from key provider ([Entrust guide](https://nshielddocs.entrust.com/interops-docs/vmware-vsphere-kc/procedures.html)) | Per VM and disk | Datastore-level access, backup copies | Guest-level access |
| vMotion encryption | Per VM setting | In flight | Network capture | None |
| Application TLS | Avi termination, mTLS inside mesh | API paths | Eavesdropping | Endpoint compromise |

- Enabling vSAN encryption on an existing cluster triggers a rolling reformat of all disk groups. Plan capacity headroom and a maintenance window ([vSAN admin guide](https://docs.vmware.com/en/VMware-vSphere/8.0/vsan-administration/GUID-E7CA36B7-D7EB-423A-ADD1-7E410E36F5A7.html)).
- The docs list a Wipe residual data option for devices that hold prior data. Use it when re-purposing hardware.
- VM Encryption on vGPU and passthrough VMs: confirm support and vMotion behavior for the exact vSphere, driver and GPU combination \[V\].
- Audit check: the PowerCLI test `Get-Cluster | Get-VsanClusterConfiguration | Select Name,EncryptionEnabled` is in the DISA STIG for vSphere 8. Run it as a scheduled compliance job.

**7.3 Key architecture**

- External KMS with HSM-backed root for Tier 1-2. Native Key Provider for Tier 0 only, if the FI accepts it. Confirm HSM options for NKP \[V\].
- One key provider per tier. Tier 2 uses a key provider per use case or business unit so a single key destruction is scoped.
- Key backup, dual control, rotation schedule, and break-glass access documented in the FI key management standard.
- KMS availability is a dependency of VM boot and vSAN. Treat the KMS as Tier 2 infrastructure. Size its HA and DR.
- Key provider audit logs go to VCF Operations for logs and SIEM.

**7.4 Destruction and sanitisation (5.4e, 5.26)**

Clause 5.4(e) asks for timely destruction of training data, model artefacts and outputs once decommissioned. Cryptographic erase works only where the key scope equals the data scope.

| Scenario | Method | Scope check | Evidence |
| --- | --- | --- | --- |
| One model or VM retired | Delete VM. If VM Encryption was used with a use-case key, destroy that key after all copies are gone | Key must protect only that use case | Key destruction log, deletion record |
| Cluster retired | Destroy cluster key at KMS. Reformat devices. Vendor secure erase or physical destruction for media leaving the site | Cluster key protects the entire datastore | Destruction certificate |
| Kubernetes volumes | Delete PVC with reclaim policy Delete. Storage class carries encryption policy. Confirm vSAN object removal | PVs removed and no retained volumes | Volume list diff |
| Model artifacts | Delete OCI artifact in Harbor, then run Harbor garbage collection to free blobs. Purge replicas and mirrored registries | Registry replication targets included | GC job log |
| Backups, snapshots, DR replicas | Enumerate every copy. Expire. Destroy backup keys | Copy inventory complete | Copy inventory and expiry log |
| Prompts, responses, logs | Retention policy in log platform. Expiry job | Classification equals source data | Retention config export |

- State in the FI data destruction standard whether cryptographic erase is accepted as sanitisation. MAS requires timely destruction or sanitisation, not a method.
- Do not describe cluster key deletion as dataset erasure. It is not granular.

**7.5 vSAN File Services for datasets**

- One file share per data class and tenant. NFS exports limited to the node pool IPs of authorized VKS clusters.
- Kerberos for NFS 4.1 or SMB with AD where identity is needed. Identity is still node-level for CSI mounts.
- Container-level control comes from namespace RBAC, PV binding rules and NetworkPolicy.
- Read-only mounts for training pipelines that must not modify source data.
- Snapshots of datasets are version evidence for 5.4(g) and 5.17.
