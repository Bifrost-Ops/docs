# Secrets, Terraform state and backup security

## Rules
- No plaintext passwords, API tokens, kubeconfigs, private keys, Terraform state or backup archives in public Git.
- Git may contain **encrypted secret manifests only** if encryption key management is external and recovery is proven.
- Never rely on the management cluster to decrypt the only copy of its own bootstrap secrets.
- Terraform state is sensitive even when no explicit secrets were configured; treat all snapshots as confidential.

## Reference design: on-prem
- External Vault-compatible secret store **or** SOPS-encrypted Git secrets using an externally escrowed age/KMS key. Choose one per environment and document key recovery.
- Versioned, encrypted, access-controlled object storage for Terraform state, with a locking mechanism supported by the selected backend and tested restore.
- Independent encrypted backup repository for K3s snapshots, server tokens, PVC/application data and APIC backups.
- Offsite/immutable copy with distinct credentials and recovery access.

## Reference design: cloud
- Cloud secret manager/KMS with scoped workload identities and audited break-glass.
- Terraform backend using supported state locking, server-side encryption and versioning.
- Backup storage with immutable retention, cross-account or cross-region copies where appropriate.
- **Do not assume** an on-prem ACI tenant extends natively into cloud networking.

## Secret lifecycle
1. Create secret outside Git with approved owner, scope and rotation period.
2. Grant minimal access to the intended controller/cluster.
3. Validate audit events and ensure logs do not leak values.
4. Back up encryption keys/recovery shares independently.
5. Test a clean restore and rotation/revocation procedure.
6. On suspected compromise, revoke/rotate credentials and audit downstream dependencies.

## Terraform state lifecycle
1. Initialize remote backend and verify encryption, locking, versioning and access.
2. Use separate state per fabric, tenant and environment ownership boundary.
3. Back up state before changes and preserve backend versions.
4. For recovery, stop writers, restore a known-good version, check lineage/serial and reconcile actual infrastructure.
5. Never assume restoring an old state snapshot changes live APIC resources.

## Failure tests
- Secret store offline: can bootstrap a replacement using documented external recovery assets?
- KMS key lost: is there an approved, independently protected recovery path?
- State backend unavailable: can state be restored without creating conflicting writers?
- GitHub unavailable: can a verified mirror supply the deployment revision?
- Management cluster lost: can recovery operators reach all required systems?

## Decision log
Select and test specific products and versions before implementation; this document deliberately avoids inventing a credential storage backend.
