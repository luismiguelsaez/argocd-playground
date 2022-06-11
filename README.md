
## Issues

### Error while applying Prometheus stack CRDS

- Description: due to limitiatios in annotations length, one of the CRDs is failing to be created
  ```
  Failed sync attempt to 8.0.6: one or more objects failed to apply, reason: CustomResourceDefinition.apiextensions.k8s.io "prometheuses.monitoring.coreos.com" is invalid: metadata.annotations: Too long: must have at most 262144 bytes,unable to recognize "/dev/shm/846913923": no matches for kind "Prometheus" in version "monitoring.coreos.com/v1" (retried 5 times).
  ```

- Affected helm charts

  - prometheus-community/kube-prometheus-stack ( tested version 36.0.0 )
  - bitnami/kube-prometheus ( tested version 8.0.6 )

- Github issues
  - https://github.com/argoproj/argo-cd/issues/2267
  - https://github.com/prometheus-community/helm-charts/issues/1500
