# OpenShift RBAC Enforcement with ACM + OpenShift GitOps

This repo contains:
- **rbac/**: Namespace Roles/RoleBindings for `default`, `appA`, `appB`
- **hub/openshift-gitops/**: ACM Placement + GitOpsCluster + ApplicationSet glue (apply on the hub cluster)
- **hub/policies/**: ACM Governance Policy + Placement/Binding to enforce and report drift

## Quick start (hub cluster)

```bash
# 1) Label the managed (guest) clusters you want governed
oc label managedcluster <guest1> rbac=baseline --overwrite
oc label managedcluster <guest2> rbac=baseline --overwrite

# 2) Apply ACM↔GitOps wiring and permissions
oc apply -f hub/openshift-gitops/

# 3) Apply ACM governance policy (optional but recommended)
oc apply -f hub/policies/
```

## ArgoCD ApplicationSet
- Edit `hub/openshift-gitops/appset-rbac-baseline.yaml` and set `repoURL` to your Git remote.
- The ApplicationSet syncs the `rbac/` folder to every cluster selected by the ACM Placement.

## Notes
- We **do not** create the `default` namespace; we only bind roles in it.
- Custom Roles are least-privilege: `ops` includes Secret access, `developer` excludes Secrets, `readonly` is read-only without Secrets.
- ACM Governance `Policy` asserts that RoleBindings and Roles exist and match (auto-remediate with `remediationAction: enforce`).
