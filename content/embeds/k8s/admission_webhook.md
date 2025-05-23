```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  labels:
    app: redis-enterprise
  name: redis-enterprise-admission
webhooks:
  - admissionReviewVersions:
      - v1beta1
    clientConfig:
      service:
        name: admission
        path: /admission
        namespace: OPERATOR_NAMESPACE
      caBundle: "" # Fill in with BASE64 encoded signed cert
    failurePolicy: Fail
    matchPolicy: Exact
    name: redisenterprise.admission.redislabs
    rules:
      - apiGroups:
          - app.redislabs.com
        apiVersions:
          - v1alpha1
        operations:
          - CREATE
          - UPDATE
        resources:
          - redisenterprisedatabases
          - redisenterpriseactiveactivedatabases
          - redisenterpriseremoteclusters
    sideEffects: None
    timeoutSeconds: 30
```
