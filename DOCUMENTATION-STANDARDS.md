# Documentation and YAML standards

## Required YAML comments
Every meaningful configurable section must explain:
1. **Intent:** what the setting controls and why it exists.
2. **Scope:** environment, device/cluster and affected traffic.
3. **Dependencies:** objects, networks, services and ordering constraints.
4. **Change risk:** impact of changing/removing the value.
5. **Recovery:** how to revert or rebuild safely.

Prefer a concise comment directly above the key. Keep comments current when changing values. Use clear examples without real credentials.

## Required runbook structure
- Purpose and architecture context
- Prerequisites and supported versions
- Inputs, secret sources and defaults
- Step-by-step deployment with verification after each step
- Expected outputs and failure symptoms
- Rollback and full-rebuild procedures
- Destructive operation warnings
- Troubleshooting and known limitations

## Example
```yaml
# Environment identifier used to select the correct state and change approvals.
# Changing this value must NOT implicitly target another APIC or state backend.
environment: lab

# Logical VRF name. Must be unique within the intended ACI tenant.
# Renaming a managed VRF may cause replacement; inspect the Terraform plan.
vrf_name: LAB-VRF
```

## Definition of done
A configuration change is incomplete without updated comments, validation, recovery instructions and a record of how to verify the result.
