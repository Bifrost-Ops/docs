# Recovery asset register — template

Maintain a **non-secret** copy per environment. Actual credentials, keys and backups live in approved external storage.

| Asset | Storage location reference | Encryption/key owner | Backup frequency | Restore test |
| --- | --- | --- | --- | --- |
| Git repositories and release tags | TBD | Repo administrator | After release | TBD |
| Terraform state: fabric | TBD | Infrastructure owner | Versioned every change | TBD |
| Terraform state: tenants | TBD | Infrastructure owner | Versioned every change | TBD |
| APIC config backup | TBD | Fabric operator | Before changes + schedule | TBD |
| Management K3s datastore snapshot + token | TBD | Platform operator | Scheduled | TBD |
| Service K3s datastore snapshot + token | TBD | Platform operator | Scheduled | TBD |
| Application/PV backup | TBD | App owner | Per RPO | TBD |
| Secrets and decryption recovery | TBD | Security owner | After changes | TBD |
| DNS/NTP/bootstrap configuration | TBD | Platform operator | Every change | TBD |
| Images and pinned artifact inventory | TBD | Platform operator | Each tested release | TBD |

## Release recovery manifest
For each tested deployment, record:
- Git commit SHA of all five repositories
- K3s/Cilium/Argo CD/Helm/provider/NAC versions
- Terraform state backend and version IDs (not state contents)
- Backup IDs and encryption key references (not secret values)
- APIC backup reference and controller version
- Verified restore date, operator, measured RPO/RTO and unresolved gaps

Never mark a package `recovery-tested` until the entire restore path has been exercised.
