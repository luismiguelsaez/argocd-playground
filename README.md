
## Argo Rollouts

- Docs
  - https://argoproj.github.io/argo-rollouts

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

- Proposals and PRs
  - https://github.com/argoproj/argo-cd/pull/8123
  - https://github.com/argoproj/gitops-engine/pull/363


## Argo canary test

- Modify image version in `overlays/minikube/flask-api-values`. Three images available
  - `flask-api:v2` - 100% of HTTP 200 responses
  - `flask-api:v2-success` - 95% of HTTP 200 responses
  - `flask-api:v2-fail` - 5% of HTTP 200 responses and 95% of HTTP 500 responses

- Check rollout status in [rollouts UI](http://argo-rollouts.minikube.cloud/rollout/flask-api)

- Status can be seen in `rollouts controller` pods as well
  ```bash
  klf -n argocd -l app.kubernetes.io/component=rollouts-controller
  ```
  ```
  time="2022-06-25T16:11:12Z" level=info msg="Event(v1.ObjectReference{Kind:\"Rollout\", Namespace:\"examples\", Name:\"flask-api\", UID:\"72458bd4-7f6c-4027-bb59-6b5c00633b80\", APIVersion:\"argoproj.io/v1alpha1\", ResourceVersion:\"7404\", FieldPath:\"\"}): type: 'Warning' reason: 'RolloutAborted' Rollout aborted update to revision 6: Metric \"success-rate\" assessed Error due to consecutiveErrors (5) > consecutiveErrorLimit (4): \"Error Message: invalid operation: >= (mismatched types float64 and string)\""
  ```
  ```
  time="2022-06-25T16:37:09Z" level=info msg="Running overdue measurement" analysisrun=flask-api-cb57d55dd-8 metric=success-rate namespace=examples
  time="2022-06-25T16:37:09Z" level=info msg="Measurement Completed. Result: Failed" analysisrun=flask-api-cb57d55dd-8 metric=success-rate namespace=examples
  ```

- Check metrics in [prometheus](http://prometheus.minikube.cloud/graph?g0.expr=(%20sum(irate(api_requests_latency_by_status_sum%7Bservice%3D%22flask-api-canary%22%2Cstatus%3D%22200%22%7D%5B5m%5D))%20%2F%20sum(irate(api_requests_latency_by_status_sum%7Bservice%3D%22flask-api-canary%22%7D%5B5m%5D))%20)%20*%20100&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h)
