```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app: redis-enterprise
  name: redis-enterprise-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: redis-enterprise-operator
subjects:
  - kind: ServiceAccount
    name: redis-enterprise-operator
```
