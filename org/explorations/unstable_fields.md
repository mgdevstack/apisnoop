- [calculating fields](#sec-1)
- [alpha/beta, deprecated and feature gated fields](#sec-2)


# calculating fields<a id="sec-1"></a>

-   **required:** <span class="underline">boolean</span> : checks field exist in required section of schema
-   **deprecated:** <span class="underline">boolean</span> : description containing 'deprecated' cases insensitive
-   **release:** <span class="underline">alpha, beta, or ga</span> : a very specific description search :
    
    See <https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md#alpha-field-in-existing-api-version>
    
    ```sql-mode
    CASE
    WHEN (   description ilike '%This field is alpha-level%'
          or description ilike '%This is an alpha field%'
          or description ilike '%This is an alpha feature%') THEN 'alpha'
    WHEN (   description ilike '%This field is beta-level%'
          or description ilike '%This field is beta%'
          or description ilike '%This is a beta feature%'
          or description ilike '%This is an beta feature%'
          or description ilike '%This is an beta field%') THEN 'beta'
    ELSE 'ga'
    END AS release,
    ```
-   **feature gated:** <span class="underline">boolean</span> : a very specific description search
    
    ```sql-mode
    CASE
    WHEN (   description ilike '%requires the % feature gate to be enabled%'
          or description ilike '%depends on the % feature gate being enabled%'
          or description ilike '%requires the % feature flag to be enabled%'
          or description ilike '%honored if the API server enables the % feature gate%'
          or description ilike '%honored by servers that enable the % feature%'
          or description ilike '%requires enabling % feature gate%'
          or description ilike '%honored by clusters that enables the % feature%'
          or description ilike '%only if the % feature gate is enabled%'
    ) THEN true
    ELSE false
                 END AS feature_gated,
    ```

# alpha/beta, deprecated and feature gated fields<a id="sec-2"></a>

-   Is this list complete?
-   If not is the way we calculate them above incorrect?
-   With this list, should we try to add metadata to the openapi spec itself?

```sql-mode
select
  release as rel,
  required as req,
  deprecated as depr,
  feature_gated as feat,
  field_schema,
  field_name,
  field_kind
  from api_schema_field
 where
 release = 'alpha'
 or release = 'beta'
 or deprecated
 or feature_gated
 order by release, depr, feat,
          length(field_schema),
          field_schema, field_name;
```

```sql-mode
  rel  | req | depr | feat |                                         field_schema                                          |          field_name           |                                          field_kind                                          
-------|-----|------|------|-----------------------------------------------------------------------------------------------|-------------------------------|----------------------------------------------------------------------------------------------
 alpha | f   | f    | f    | io.k8s.api.storage.v1.VolumeAttachmentSource                                                  | inlineVolumeSpec              | io.k8s.api.core.v1.PersistentVolumeSpec
 alpha | f   | f    | f    | io.k8s.api.storage.v1beta1.VolumeAttachmentSource                                             | inlineVolumeSpec              | io.k8s.api.core.v1.PersistentVolumeSpec
 alpha | f   | f    | f    | io.k8s.api.storage.v1alpha1.VolumeAttachmentSource                                            | inlineVolumeSpec              | io.k8s.api.core.v1.PersistentVolumeSpec
 alpha | f   | f    | t    | io.k8s.api.core.v1.PodSpec                                                                    | ephemeralContainers           | io.k8s.api.core.v1.EphemeralContainer
 alpha | f   | f    | t    | io.k8s.api.core.v1.PodSpec                                                                    | overhead                      | integer
 alpha | f   | f    | t    | io.k8s.api.core.v1.PodSpec                                                                    | preemptionPolicy              | string
 alpha | f   | f    | t    | io.k8s.api.core.v1.PodSpec                                                                    | topologySpreadConstraints     | io.k8s.api.core.v1.TopologySpreadConstraint
 alpha | f   | f    | t    | io.k8s.api.batch.v1.JobSpec                                                                   | ttlSecondsAfterFinished       | integer
 alpha | f   | f    | t    | io.k8s.api.core.v1.PodStatus                                                                  | ephemeralContainerStatuses    | io.k8s.api.core.v1.ContainerStatus
 alpha | f   | f    | t    | io.k8s.api.node.v1beta1.RuntimeClass                                                          | overhead                      | io.k8s.api.node.v1beta1.Overhead
 alpha | f   | f    | t    | io.k8s.api.scheduling.v1.PriorityClass                                                        | preemptionPolicy              | string
 alpha | f   | f    | t    | io.k8s.api.node.v1alpha1.RuntimeClassSpec                                                     | overhead                      | io.k8s.api.node.v1alpha1.Overhead
 alpha | f   | f    | t    | io.k8s.api.scheduling.v1beta1.PriorityClass                                                   | preemptionPolicy              | string
 alpha | f   | f    | t    | io.k8s.api.core.v1.CSIPersistentVolumeSource                                                  | controllerExpandSecretRef     | io.k8s.api.core.v1.SecretReference
 alpha | f   | f    | t    | io.k8s.api.scheduling.v1alpha1.PriorityClass                                                  | preemptionPolicy              | string
 alpha | f   | f    | t    | io.k8s.api.policy.v1beta1.PodSecurityPolicySpec                                               | allowedCSIDrivers             | io.k8s.api.policy.v1beta1.AllowedCSIDriver
 alpha | f   | f    | t    | io.k8s.api.core.v1.WindowsSecurityContextOptions                                              | gmsaCredentialSpec            | string
 alpha | f   | f    | t    | io.k8s.api.core.v1.WindowsSecurityContextOptions                                              | gmsaCredentialSpecName        | string
 alpha | f   | f    | t    | io.k8s.api.core.v1.WindowsSecurityContextOptions                                              | runAsUserName                 | string
 alpha | f   | f    | t    | io.k8s.apimachinery.pkg.apis.meta.v1.APIResource                                              | storageVersionHash            | string
 alpha | f   | f    | t    | io.k8s.api.extensions.v1beta1.PodSecurityPolicySpec                                           | allowedCSIDrivers             | io.k8s.api.extensions.v1beta1.AllowedCSIDriver
 alpha | f   | f    | t    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceConversion        | webhookClientConfig           | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.WebhookClientConfig
 alpha | f   | f    | t    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceDefinitionVersion | additionalPrinterColumns      | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceColumnDefinition
 alpha | f   | f    | t    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceDefinitionVersion | schema                        | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceValidation
 alpha | f   | f    | t    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceDefinitionVersion | subresources                  | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceSubresources
 beta  | f   | f    | f    | io.k8s.api.core.v1.PodSpec                                                                    | runtimeClassName              | string
 beta  | f   | f    | f    | io.k8s.api.core.v1.PodSpec                                                                    | shareProcessNamespace         | integer
 beta  | f   | f    | f    | io.k8s.api.core.v1.Container                                                                  | volumeDevices                 | io.k8s.api.core.v1.VolumeDevice
 beta  | f   | f    | f    | io.k8s.api.core.v1.VolumeMount                                                                | mountPropagation              | string
 beta  | f   | f    | f    | io.k8s.api.core.v1.VolumeMount                                                                | subPathExpr                   | string
 beta  | f   | f    | f    | io.k8s.api.core.v1.EphemeralContainer                                                         | volumeDevices                 | io.k8s.api.core.v1.VolumeDevice
 beta  | f   | f    | f    | io.k8s.api.core.v1.PersistentVolumeSpec                                                       | volumeMode                    | string
 beta  | f   | f    | f    | io.k8s.api.networking.v1.NetworkPolicySpec                                                    | egress                        | io.k8s.api.networking.v1.NetworkPolicyEgressRule
 beta  | f   | f    | f    | io.k8s.api.networking.v1.NetworkPolicySpec                                                    | policyTypes                   | string
 beta  | f   | f    | f    | io.k8s.api.core.v1.PersistentVolumeClaimSpec                                                  | volumeMode                    | string
 beta  | f   | f    | f    | io.k8s.api.extensions.v1beta1.NetworkPolicySpec                                               | egress                        | io.k8s.api.extensions.v1beta1.NetworkPolicyEgressRule
 beta  | f   | f    | f    | io.k8s.api.extensions.v1beta1.NetworkPolicySpec                                               | policyTypes                   | string
 ga    | f   | f    | t    | io.k8s.api.core.v1.SecurityContext                                                            | procMount                     | string
 ga    | f   | f    | t    | io.k8s.api.storage.v1.StorageClass                                                            | allowedTopologies             | io.k8s.api.core.v1.TopologySelectorTerm
 ga    | f   | f    | t    | io.k8s.api.storage.v1.StorageClass                                                            | volumeBindingMode             | string
 ga    | f   | f    | t    | io.k8s.api.storage.v1beta1.StorageClass                                                       | allowedTopologies             | io.k8s.api.core.v1.TopologySelectorTerm
 ga    | f   | f    | t    | io.k8s.api.storage.v1beta1.StorageClass                                                       | volumeBindingMode             | string
 ga    | f   | f    | t    | io.k8s.api.core.v1.PersistentVolumeClaimSpec                                                  | dataSource                    | io.k8s.api.core.v1.TypedLocalObjectReference
 ga    | f   | f    | t    | io.k8s.api.policy.v1beta1.PodSecurityPolicySpec                                               | allowedProcMountTypes         | string
 ga    | f   | f    | t    | io.k8s.api.policy.v1beta1.PodSecurityPolicySpec                                               | runAsGroup                    | io.k8s.api.policy.v1beta1.RunAsGroupStrategyOptions
 ga    | f   | f    | t    | io.k8s.api.policy.v1beta1.PodSecurityPolicySpec                                               | runtimeClass                  | io.k8s.api.policy.v1beta1.RuntimeClassStrategyOptions
 ga    | f   | f    | t    | io.k8s.api.extensions.v1beta1.PodSecurityPolicySpec                                           | allowedProcMountTypes         | string
 ga    | f   | f    | t    | io.k8s.api.extensions.v1beta1.PodSecurityPolicySpec                                           | runAsGroup                    | io.k8s.api.extensions.v1beta1.RunAsGroupStrategyOptions
 ga    | f   | f    | t    | io.k8s.api.extensions.v1beta1.PodSecurityPolicySpec                                           | runtimeClass                  | io.k8s.api.extensions.v1beta1.RuntimeClassStrategyOptions
 ga    | f   | t    | f    | io.k8s.api.core.v1.Volume                                                                     | gitRepo                       | io.k8s.api.core.v1.GitRepoVolumeSource
 ga    | f   | t    | f    | io.k8s.api.core.v1.PodSpec                                                                    | serviceAccount                | string
 ga    | f   | t    | f    | io.k8s.api.core.v1.NodeSpec                                                                   | externalID                    | string
 ga    | f   | t    | f    | io.k8s.api.core.v1.NodeStatus                                                                 | phase                         | string
 ga    | f   | t    | f    | io.k8s.api.core.v1.EventSeries                                                                | state                         | string
 ga    | f   | t    | f    | io.k8s.api.events.v1beta1.Event                                                               | deprecatedCount               | integer
 ga    | f   | t    | f    | io.k8s.api.events.v1beta1.Event                                                               | deprecatedFirstTimestamp      | io.k8s.apimachinery.pkg.apis.meta.v1.Time
 ga    | f   | t    | f    | io.k8s.api.events.v1beta1.Event                                                               | deprecatedLastTimestamp       | io.k8s.apimachinery.pkg.apis.meta.v1.Time
 ga    | f   | t    | f    | io.k8s.api.events.v1beta1.Event                                                               | deprecatedSource              | io.k8s.api.core.v1.EventSource
 ga    | t   | t    | f    | io.k8s.api.events.v1beta1.EventSeries                                                         | state                         | string
 ga    | f   | t    | f    | io.k8s.api.apps.v1beta1.DeploymentSpec                                                        | rollbackTo                    | io.k8s.api.apps.v1beta1.RollbackConfig
 ga    | f   | t    | f    | io.k8s.api.core.v1.FlockerVolumeSource                                                        | datasetName                   | string
 ga    | f   | t    | f    | io.k8s.api.core.v1.PersistentVolumeSpec                                                       | persistentVolumeReclaimPolicy | string
 ga    | f   | t    | f    | io.k8s.api.extensions.v1beta1.DaemonSetSpec                                                   | templateGeneration            | integer
 ga    | f   | t    | f    | io.k8s.api.extensions.v1beta1.DeploymentSpec                                                  | rollbackTo                    | io.k8s.api.extensions.v1beta1.RollbackConfig
 ga    | f   | t    | f    | io.k8s.apimachinery.pkg.apis.meta.v1.ListMeta                                                 | selfLink                      | string
 ga    | f   | t    | f    | io.k8s.apimachinery.pkg.apis.meta.v1.ObjectMeta                                               | selfLink                      | string
 ga    | f   | t    | f    | io.k8s.apimachinery.pkg.apis.meta.v1.DeleteOptions                                            | orphanDependents              | integer
 ga    | f   | t    | f    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1.CustomResourceDefinitionSpec         | preserveUnknownFields         | integer
 ga    | f   | t    | f    | io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1beta1.CustomResourceDefinitionSpec    | version                       | string
(69 rows)

```
