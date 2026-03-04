# Kubernetes DRA Migration Notes (Intel iGPU)

## Current status

- Cluster: Kubernetes `v1.35.x`.
- Current GPU allocation path: Intel device plugin (`gpu.intel.com/i915`).
- Workload dependency in this repo: `kubernetes/apps/media/jellyfin/jellyfin.yaml`.

## Recommendation

- Keep Intel device plugin as default production path for now.
- Evaluate DRA using a controlled pilot first (single non-critical workload).
- Migrate only after validating stability and operational simplicity.

## Why

- Kubernetes DRA is now stable, but vendor-specific GPU DRA drivers and ecosystem maturity still require careful validation for home-lab production use.
- Intel's DRA-related stack is available but still marked beta in upstream driver repositories, so risk is higher than the current plugin path.

## Suggested pilot scope

1. Keep current Jellyfin deployment unchanged (`gpu.intel.com/i915`).
2. Stand up a separate test workload in a dedicated namespace.
3. Validate:
   - device discovery and allocation
   - pod restart behavior
   - node reboot behavior
   - coexistence with existing plugin-managed workloads
4. If stable for multiple days, plan staged migration.

## Upstream references

- Kubernetes DRA docs: <https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/>
- Intel resource drivers repo (DRA-related): <https://github.com/intel/intel-resource-drivers-for-kubernetes>
- Intel device plugins repo: <https://github.com/intel/intel-device-plugins-for-kubernetes>
