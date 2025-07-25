<!--- app-name: Neo4j -->

# Bitnami package for Neo4j

Neo4j is a high performance graph store with all the features expected of a mature and robust database, like a friendly query language and ACID transactions.

[Overview of Neo4j](http://www.neo4j.org/)

Trademarks: This software listing is packaged by Bitnami. The respective trademarks mentioned in the offering are owned by the respective companies, and use of them does not imply any affiliation or endorsement.

## TL;DR

```console
helm install my-release oci://registry-1.docker.io/bitnamicharts/neo4j
```

Looking to use neo4j in production? Try [VMware Tanzu Application Catalog](https://bitnami.com/enterprise), the commercial edition of the Bitnami catalog.

## ⚠️ Important Notice: Upcoming changes to the Bitnami Catalog

Beginning August 28th, 2025, Bitnami will evolve its public catalog to offer a curated set of hardened, security-focused images under the new [Bitnami Secure Images initiative](https://news.broadcom.com/app-dev/broadcom-introduces-bitnami-secure-images-for-production-ready-containerized-applications). As part of this transition:

- Granting community users access for the first time to security-optimized versions of popular container images.
- Bitnami will begin deprecating support for non-hardened, Debian-based software images in its free tier and will gradually remove non-latest tags from the public catalog. As a result, community users will have access to a reduced number of hardened images. These images are published only under the “latest” tag and are intended for development purposes
- Starting August 28th, over two weeks, all existing container images, including older or versioned tags (e.g., 2.50.0, 10.6), will be migrated from the public catalog (docker.io/bitnami) to the “Bitnami Legacy” repository (docker.io/bitnamilegacy), where they will no longer receive updates.
- For production workloads and long-term support, users are encouraged to adopt Bitnami Secure Images, which include hardened containers, smaller attack surfaces, CVE transparency (via VEX/KEV), SBOMs, and enterprise support.

These changes aim to improve the security posture of all Bitnami users by promoting best practices for software supply chain integrity and up-to-date deployments. For more details, visit the [Bitnami Secure Images announcement](https://github.com/bitnami/containers/issues/83267).

## Introduction

Bitnami charts for Helm are carefully engineered, actively maintained and are the quickest and easiest way to deploy containers on a Kubernetes cluster that are ready to handle production workloads.

This chart bootstraps a [Neo4j](https://github.com/bitnami/containers/tree/main/bitnami/neo4j) deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8.0+
- PV provisioner support in the underlying infrastructure
- ReadWriteMany volumes for deployment scaling

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release oci://REGISTRY_NAME/REPOSITORY_NAME/neo4j
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

The command deploys neo4j on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Configuration and installation details

### [Rolling VS Immutable tags](https://techdocs.broadcom.com/us/en/vmware-tanzu/application-catalog/tanzu-application-catalog/services/tac-doc/apps-tutorials-understand-rolling-tags-containers-index.html)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

### Update credentials

Bitnami charts configure credentials at first boot. Any further change in the secrets or credentials require manual intervention. Follow these instructions:

- Update the user password following [the upstream documentation](https://neo4j.com/docs/operations-manual/current/authentication-authorization/password-and-user-recovery/)
- Update the password secret with the new values (replace the SECRET_NAME and PASSWORD placeholders)

```shell
kubectl create secret generic SECRET_NAME --from-literal=password=PASSWORD --dry-run -o yaml | kubectl apply -f -
```

### Ingress

This chart provides support for Ingress resources. If you have an ingress controller installed on your cluster, such as [nginx-ingress-controller](https://github.com/bitnami/charts/tree/main/bitnami/nginx-ingress-controller) or [contour](https://github.com/bitnami/charts/tree/main/bitnami/contour) you can utilize the ingress controller to serve your application.To enable Ingress integration, set `ingress.enabled` to `true`.

The most common scenario is to have one host name mapped to the deployment. In this case, the `ingress.hostname` property can be used to set the host name. The `ingress.tls` parameter can be used to add the TLS configuration for this host.

However, it is also possible to have more than one host. To facilitate this, the `ingress.extraHosts` parameter (if available) can be set with the host names specified as an array. The `ingress.extraTLS` parameter (if available) can also be used to add the TLS configuration for extra hosts.

> NOTE: For each host specified in the `ingress.extraHosts` parameter, it is necessary to set a name, path, and any annotations that the Ingress controller should know about. Not all annotations are supported by all Ingress controllers, but [this annotation reference document](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md) lists the annotations supported by many popular Ingress controllers.

Adding the TLS parameter (where available) will cause the chart to generate HTTPS URLs, and the  application will be available on port 443. The actual TLS secrets do not have to be generated by this chart. However, if TLS is enabled, the Ingress record will not work until the TLS secret exists.

[Learn more about Ingress controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

### Securing traffic using TLS

TLS support can be enabled in the chart by specifying the `tls.` parameters while creating a release. The following parameters should be configured to properly enable the TLS support in the cluster:

- `tls.enabled`: Enable TLS support. Defaults to `false`
- `tls.existingSecret`: Name of the secret that contains the certificates. No defaults.
- `tls.certFilename`: Certificate filename. No defaults.
- `tls.certKeyFilename`: Certificate key filename. No defaults.
- `tls.certCAFilename`: CA Certificate filename. No defaults.

For example:

First, create the secret with the certificates files:

```console
kubectl create secret generic certificates-tls-secret --from-file=./cert.pem --from-file=./cert.key --from-file=./ca.pem
```

Then, use the following parameters:

```console
tls.enabled="true"
tls.existingSecret="certificates-tls-secret"
tls.certFilename="cert.pem"
tls.certKeyFilename="cert.key"
tls.certCAFilename="ca.pem"
```

### Additional environment variables

In case you want to add extra environment variables (useful for advanced operations like custom init scripts), you can use the `extraEnvVars` property.

```yaml
extraEnvVars:
  - name: LOG_LEVEL
    value: error
```

Alternatively, you can use a ConfigMap or a Secret with the environment variables. To do so, use the `extraEnvVarsCM` or the `extraEnvVarsSecret` values.

### Sidecars

If additional containers are needed in the same pod as neo4j (such as additional metrics or logging exporters), they can be defined using the `sidecars` parameter.

```yaml
sidecars:
- name: your-image-name
  image: your-image
  imagePullPolicy: Always
  ports:
  - name: portname
    containerPort: 1234
```

If these sidecars export extra ports, extra port definitions can be added using the `service.extraPorts` parameter (where available), as shown in the example below:

```yaml
service:
  extraPorts:
  - name: extraPort
    port: 11311
    targetPort: 11311
```

> NOTE: This Helm chart already includes sidecar containers for the Prometheus exporters (where applicable). These can be activated by adding the `--enable-metrics=true` parameter at deployment time. The `sidecars` parameter should therefore only be used for any extra sidecar containers.

If additional init containers are needed in the same pod, they can be defined using the `initContainers` parameter. Here is an example:

```yaml
initContainers:
  - name: your-image-name
    image: your-image
    imagePullPolicy: Always
    ports:
      - name: portname
        containerPort: 1234
```

Learn more about [sidecar containers](https://kubernetes.io/docs/concepts/workloads/pods/) and [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/).

### Pod affinity

This chart allows you to set your custom affinity using the `affinity` parameter. Find more information about Pod affinity in the [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

As an alternative, use one of the preset configurations for pod affinity, pod anti-affinity, and node affinity available at the [bitnami/common](https://github.com/bitnami/charts/tree/main/bitnami/common#affinities) chart. To do so, set the `podAffinityPreset`, `podAntiAffinityPreset`, or `nodeAffinityPreset` parameters.

### Backup and restore

To back up and restore Helm chart deployments on Kubernetes, you need to back up the persistent volumes from the source deployment and attach them to a new deployment using [Velero](https://velero.io/), a Kubernetes backup/restore tool. Find the instructions for using Velero in [this guide](https://techdocs.broadcom.com/us/en/vmware-tanzu/application-catalog/tanzu-application-catalog/services/tac-doc/apps-tutorials-backup-restore-deployments-velero-index.html).

## Persistence

The [Bitnami neo4j](https://github.com/bitnami/containers/tree/main/bitnami/neo4j) image stores the neo4j data and configurations at the `/bitnami` path of the container. Persistent Volume Claims are used to keep the data across deployments.

If you encounter errors when working with persistent volumes, refer to our [troubleshooting guide for persistent volumes](https://docs.bitnami.com/kubernetes/faq/troubleshooting/troubleshooting-persistence-volumes/).

## Parameters

### Global parameters

| Name                                                  | Description                                                                                                                                                                                                                                                                                                                                                         | Value   |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `global.imageRegistry`                                | Global Docker image registry                                                                                                                                                                                                                                                                                                                                        | `""`    |
| `global.imagePullSecrets`                             | Global Docker registry secret names as an array                                                                                                                                                                                                                                                                                                                     | `[]`    |
| `global.defaultStorageClass`                          | Global default StorageClass for Persistent Volume(s)                                                                                                                                                                                                                                                                                                                | `""`    |
| `global.storageClass`                                 | DEPRECATED: use global.defaultStorageClass instead                                                                                                                                                                                                                                                                                                                  | `""`    |
| `global.security.allowInsecureImages`                 | Allows skipping image verification                                                                                                                                                                                                                                                                                                                                  | `false` |
| `global.compatibility.openshift.adaptSecurityContext` | Adapt the securityContext sections of the deployment to make them compatible with Openshift restricted-v2 SCC: remove runAsUser, runAsGroup and fsGroup and let the platform use their allowed default IDs. Possible values: auto (apply if the detected running cluster is Openshift), force (perform the adaptation always), disabled (do not perform adaptation) | `auto`  |

### Common parameters

| Name                     | Description                                                                             | Value           |
| ------------------------ | --------------------------------------------------------------------------------------- | --------------- |
| `kubeVersion`            | Override Kubernetes version                                                             | `""`            |
| `apiVersions`            | Override Kubernetes API versions reported by .Capabilities                              | `[]`            |
| `nameOverride`           | String to partially override common.names.name                                          | `""`            |
| `fullnameOverride`       | String to fully override common.names.fullname                                          | `""`            |
| `namespaceOverride`      | String to fully override common.names.namespace                                         | `""`            |
| `commonLabels`           | Labels to add to all deployed objects                                                   | `{}`            |
| `commonAnnotations`      | Annotations to add to all deployed objects                                              | `{}`            |
| `clusterDomain`          | Kubernetes cluster domain name                                                          | `cluster.local` |
| `extraDeploy`            | Array of extra objects to deploy with the release                                       | `[]`            |
| `usePasswordFiles`       | Mount credentials as files instead of using environment variables                       | `true`          |
| `diagnosticMode.enabled` | Enable diagnostic mode (all probes will be disabled and the command will be overridden) | `false`         |
| `diagnosticMode.command` | Command to override all containers in the chart release                                 | `["sleep"]`     |
| `diagnosticMode.args`    | Args to override all containers in the chart release                                    | `["infinity"]`  |

### neo4j Parameters

| Name                                                | Description                                                                                                                                                                                                      | Value                   |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| `image.registry`                                    | neo4j image registry                                                                                                                                                                                             | `REGISTRY_NAME`         |
| `image.repository`                                  | neo4j image repository                                                                                                                                                                                           | `REPOSITORY_NAME/neo4j` |
| `image.digest`                                      | neo4j image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag image tag (immutable tags are recommended)                                                                 | `""`                    |
| `image.pullPolicy`                                  | neo4j image pull policy                                                                                                                                                                                          | `IfNotPresent`          |
| `image.pullSecrets`                                 | neo4j image pull secrets                                                                                                                                                                                         | `[]`                    |
| `image.debug`                                       | Enable neo4j image debug mode                                                                                                                                                                                    | `false`                 |
| `containerPorts.bolt`                               | neo4j Bolt container port                                                                                                                                                                                        | `7687`                  |
| `containerPorts.http`                               | neo4j HTTP container port                                                                                                                                                                                        | `7474`                  |
| `containerPorts.https`                              | neo4j HTTPS container port                                                                                                                                                                                       | `7473`                  |
| `extraContainerPorts`                               | Optionally specify extra list of additional ports for neo4j containers                                                                                                                                           | `[]`                    |
| `livenessProbe.enabled`                             | Enable livenessProbe on neo4j containers                                                                                                                                                                         | `true`                  |
| `livenessProbe.initialDelaySeconds`                 | Initial delay seconds for livenessProbe                                                                                                                                                                          | `10`                    |
| `livenessProbe.periodSeconds`                       | Period seconds for livenessProbe                                                                                                                                                                                 | `5`                     |
| `livenessProbe.timeoutSeconds`                      | Timeout seconds for livenessProbe                                                                                                                                                                                | `10`                    |
| `livenessProbe.failureThreshold`                    | Failure threshold for livenessProbe                                                                                                                                                                              | `20`                    |
| `livenessProbe.successThreshold`                    | Success threshold for livenessProbe                                                                                                                                                                              | `1`                     |
| `readinessProbe.enabled`                            | Enable readinessProbe on neo4j containers                                                                                                                                                                        | `true`                  |
| `readinessProbe.initialDelaySeconds`                | Initial delay seconds for readinessProbe                                                                                                                                                                         | `10`                    |
| `readinessProbe.periodSeconds`                      | Period seconds for readinessProbe                                                                                                                                                                                | `5`                     |
| `readinessProbe.timeoutSeconds`                     | Timeout seconds for readinessProbe                                                                                                                                                                               | `10`                    |
| `readinessProbe.failureThreshold`                   | Failure threshold for readinessProbe                                                                                                                                                                             | `20`                    |
| `readinessProbe.successThreshold`                   | Success threshold for readinessProbe                                                                                                                                                                             | `1`                     |
| `startupProbe.enabled`                              | Enable startupProbe on neo4j containers                                                                                                                                                                          | `false`                 |
| `startupProbe.initialDelaySeconds`                  | Initial delay seconds for startupProbe                                                                                                                                                                           | `10`                    |
| `startupProbe.periodSeconds`                        | Period seconds for startupProbe                                                                                                                                                                                  | `5`                     |
| `startupProbe.timeoutSeconds`                       | Timeout seconds for startupProbe                                                                                                                                                                                 | `10`                    |
| `startupProbe.failureThreshold`                     | Failure threshold for startupProbe                                                                                                                                                                               | `20`                    |
| `startupProbe.successThreshold`                     | Success threshold for startupProbe                                                                                                                                                                               | `1`                     |
| `customLivenessProbe`                               | Custom livenessProbe that overrides the default one                                                                                                                                                              | `{}`                    |
| `customReadinessProbe`                              | Custom readinessProbe that overrides the default one                                                                                                                                                             | `{}`                    |
| `customStartupProbe`                                | Custom startupProbe that overrides the default one                                                                                                                                                               | `{}`                    |
| `resourcesPreset`                                   | Set neo4j container resources according to one common preset (allowed values: none, nano, small, medium, large, xlarge, 2xlarge). This is ignored if resources is set (resources is recommended for production). | `small`                 |
| `resources`                                         | Set neo4j container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                          | `{}`                    |
| `podSecurityContext.enabled`                        | Enable neo4j pods' Security Context                                                                                                                                                                              | `true`                  |
| `podSecurityContext.fsGroupChangePolicy`            | Set filesystem group change policy for neo4j pods                                                                                                                                                                | `Always`                |
| `podSecurityContext.sysctls`                        | Set kernel settings using the sysctl interface for neo4j pods                                                                                                                                                    | `[]`                    |
| `podSecurityContext.supplementalGroups`             | Set filesystem extra groups for neo4j pods                                                                                                                                                                       | `[]`                    |
| `podSecurityContext.fsGroup`                        | Set fsGroup in neo4j pods' Security Context                                                                                                                                                                      | `1001`                  |
| `containerSecurityContext.enabled`                  | Enabled neo4j container' Security Context                                                                                                                                                                        | `true`                  |
| `containerSecurityContext.seLinuxOptions`           | Set SELinux options in neo4j container                                                                                                                                                                           | `{}`                    |
| `containerSecurityContext.runAsUser`                | Set runAsUser in neo4j container' Security Context                                                                                                                                                               | `1001`                  |
| `containerSecurityContext.runAsGroup`               | Set runAsGroup in neo4j container' Security Context                                                                                                                                                              | `1001`                  |
| `containerSecurityContext.runAsNonRoot`             | Set runAsNonRoot in neo4j container' Security Context                                                                                                                                                            | `true`                  |
| `containerSecurityContext.readOnlyRootFilesystem`   | Set readOnlyRootFilesystem in neo4j container' Security Context                                                                                                                                                  | `true`                  |
| `containerSecurityContext.privileged`               | Set privileged in neo4j container' Security Context                                                                                                                                                              | `false`                 |
| `containerSecurityContext.allowPrivilegeEscalation` | Set allowPrivilegeEscalation in neo4j container' Security Context                                                                                                                                                | `false`                 |
| `containerSecurityContext.capabilities.drop`        | List of capabilities to be dropped in neo4j container                                                                                                                                                            | `["ALL"]`               |
| `containerSecurityContext.seccompProfile.type`      | Set seccomp profile in neo4j container                                                                                                                                                                           | `RuntimeDefault`        |
| `auth.password`                                     | Password for the 'neo4j' native user                                                                                                                                                                             | `""`                    |
| `auth.existingSecret`                               | Name of a secret containing the neo4j password. Password must be stored under the 'password' key.                                                                                                                | `""`                    |
| `tls.bolt.level`                                    | Controls wether the bolt connector should enforce encrypted connections. Allowed values are: 'DISABLED', 'OPTIONAL' and 'REQUIRED'. Both 'OPTIONAL' and 'REQUIRED' would be equal to                             | `DISABLED`              |
| `tls.bolt.existingSecret`                           | Name of the secret that contains the certificates.                                                                                                                                                               | `""`                    |
| `tls.bolt.autoGenerated`                            | Automatically generate self-signed TLS certificates.                                                                                                                                                             | `false`                 |
| `tls.https.enabled`                                 | Enables Neo4j HTTPS connector                                                                                                                                                                                    | `false`                 |
| `tls.https.existingSecret`                          | Name of the secret that contains the certificates.                                                                                                                                                               | `""`                    |
| `tls.https.autoGenerated`                           | Automatically generate self-signed TLS certificates.                                                                                                                                                             | `false`                 |
| `advertisedHost`                                    | Hostname used to configure Neo4j advertised address.                                                                                                                                                             | `""`                    |
| `existingConfigmap`                                 | The name of an existing ConfigMap with your custom configuration for neo4j                                                                                                                                       | `""`                    |
| `configuration`                                     | Specify content for Neo4j config file (auto-generated based on other env. vars otherwise). Omitted if existingConfigmap is provided.                                                                             | `""`                    |
| `apocConfiguration`                                 | Specify content for Neo4j apoc config file (auto-generated based on other env. vars otherwise). Omitted if existingConfigmap is provided.                                                                        | `""`                    |
| `pluginsConfigMap`                                  | The name of an existing ConfigMap with your custom plugins for neo4j                                                                                                                                             | `""`                    |
| `command`                                           | Override default neo4j container command (useful when using custom images)                                                                                                                                       | `[]`                    |
| `args`                                              | Override default neo4j container args (useful when using custom images)                                                                                                                                          | `[]`                    |
| `automountServiceAccountToken`                      | Mount Service Account token in neo4j pods                                                                                                                                                                        | `false`                 |
| `hostAliases`                                       | neo4j pods host aliases                                                                                                                                                                                          | `[]`                    |
| `statefulsetAnnotations`                            | Annotations for neo4j statefulset                                                                                                                                                                                | `{}`                    |
| `podLabels`                                         | Extra labels for neo4j pods                                                                                                                                                                                      | `{}`                    |
| `podAnnotations`                                    | Annotations for neo4j pods                                                                                                                                                                                       | `{}`                    |
| `podAffinityPreset`                                 | Pod affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                              | `""`                    |
| `podAntiAffinityPreset`                             | Pod anti-affinity preset. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                         | `soft`                  |
| `nodeAffinityPreset.type`                           | Node affinity preset type. Ignored if `affinity` is set. Allowed values: `soft` or `hard`                                                                                                                        | `""`                    |
| `nodeAffinityPreset.key`                            | Node label key to match. Ignored if `affinity` is set                                                                                                                                                            | `""`                    |
| `nodeAffinityPreset.values`                         | Node label values to match. Ignored if `affinity` is set                                                                                                                                                         | `[]`                    |
| `affinity`                                          | Affinity for neo4j pods assignment                                                                                                                                                                               | `{}`                    |
| `nodeSelector`                                      | Node labels for neo4j pods assignment                                                                                                                                                                            | `{}`                    |
| `tolerations`                                       | Tolerations for neo4j pods assignment                                                                                                                                                                            | `[]`                    |
| `updateStrategy.type`                               | neo4j deployment strategy type                                                                                                                                                                                   | `RollingUpdate`         |
| `updateStrategy.type`                               | neo4j statefulset strategy type                                                                                                                                                                                  | `RollingUpdate`         |
| `podManagementPolicy`                               | Pod management policy for neo4j statefulset                                                                                                                                                                      | `Parallel`              |
| `priorityClassName`                                 | neo4j pods' priorityClassName                                                                                                                                                                                    | `""`                    |
| `topologySpreadConstraints`                         | Topology Spread Constraints for neo4j pod assignment spread across your cluster among failure-domains                                                                                                            | `[]`                    |
| `schedulerName`                                     | Name of the k8s scheduler (other than default) for neo4j pods                                                                                                                                                    | `""`                    |
| `terminationGracePeriodSeconds`                     | Seconds neo4j pods need to terminate gracefully                                                                                                                                                                  | `""`                    |
| `lifecycleHooks`                                    | for neo4j containers to automate configuration before or after startup                                                                                                                                           | `{}`                    |
| `extraEnvVars`                                      | Array with extra environment variables to add to neo4j containers                                                                                                                                                | `[]`                    |
| `extraEnvVarsCM`                                    | Name of existing ConfigMap containing extra env vars for neo4j containers                                                                                                                                        | `""`                    |
| `extraEnvVarsSecret`                                | Name of existing Secret containing extra env vars for neo4j containers                                                                                                                                           | `""`                    |
| `extraVolumes`                                      | Optionally specify extra list of additional volumes for the neo4j pods                                                                                                                                           | `[]`                    |
| `extraVolumeMounts`                                 | Optionally specify extra list of additional volumeMounts for the neo4j containers                                                                                                                                | `[]`                    |
| `sidecars`                                          | Add additional sidecar containers to the neo4j pods                                                                                                                                                              | `[]`                    |
| `initContainers`                                    | Add additional init containers to the neo4j pods                                                                                                                                                                 | `[]`                    |
| `pdb.create`                                        | Enable/disable a Pod Disruption Budget creation                                                                                                                                                                  | `true`                  |
| `pdb.minAvailable`                                  | Minimum number/percentage of pods that should remain scheduled                                                                                                                                                   | `""`                    |
| `pdb.maxUnavailable`                                | Maximum number/percentage of pods that may be made unavailable. Defaults to `1` if both `pdb.minAvailable` and `pdb.maxUnavailable` are empty.                                                                   | `""`                    |
| `autoscaling.vpa.enabled`                           | Enable VPA for neo4j pods                                                                                                                                                                                        | `false`                 |
| `autoscaling.vpa.annotations`                       | Annotations for VPA resource                                                                                                                                                                                     | `{}`                    |
| `autoscaling.vpa.controlledResources`               | VPA List of resources that the vertical pod autoscaler can control. Defaults to cpu and memory                                                                                                                   | `[]`                    |
| `autoscaling.vpa.maxAllowed`                        | VPA Max allowed resources for the pod                                                                                                                                                                            | `{}`                    |
| `autoscaling.vpa.minAllowed`                        | VPA Min allowed resources for the pod                                                                                                                                                                            | `{}`                    |
| `autoscaling.vpa.updatePolicy.updateMode`           | Autoscaling update policy                                                                                                                                                                                        | `Auto`                  |

### Traffic Exposure Parameters

| Name                                    | Description                                                                                                                      | Value                    |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `service.type`                          | neo4j service type                                                                                                               | `LoadBalancer`           |
| `service.ports.bolt`                    | neo4j service bolt port                                                                                                          | `7687`                   |
| `service.ports.http`                    | neo4j service HTTP port                                                                                                          | `7474`                   |
| `service.ports.https`                   | neo4j service HTTPS port                                                                                                         | `7473`                   |
| `service.nodePorts.bolt`                | Node port for bolt connector                                                                                                     | `""`                     |
| `service.nodePorts.http`                | Node port for HTTP                                                                                                               | `""`                     |
| `service.nodePorts.https`               | Node port for HTTPS                                                                                                              | `""`                     |
| `service.clusterIP`                     | neo4j service Cluster IP                                                                                                         | `""`                     |
| `service.loadBalancerIP`                | neo4j service Load Balancer IP                                                                                                   | `""`                     |
| `service.loadBalancerSourceRanges`      | neo4j service Load Balancer sources                                                                                              | `[]`                     |
| `service.externalTrafficPolicy`         | neo4j service external traffic policy                                                                                            | `Cluster`                |
| `service.annotations`                   | Additional custom annotations for neo4j service                                                                                  | `{}`                     |
| `service.extraPorts`                    | Extra ports to expose in neo4j service (normally used with the `sidecars` value)                                                 | `[]`                     |
| `service.sessionAffinity`               | Control where client requests go, to the same pod or round-robin                                                                 | `None`                   |
| `service.sessionAffinityConfig`         | Additional settings for the sessionAffinity                                                                                      | `{}`                     |
| `networkPolicy.enabled`                 | Specifies whether a NetworkPolicy should be created                                                                              | `true`                   |
| `networkPolicy.allowExternal`           | Don't require server label for connections                                                                                       | `true`                   |
| `networkPolicy.allowExternalEgress`     | Allow the pod to access any range of port and all destinations.                                                                  | `true`                   |
| `networkPolicy.addExternalClientAccess` | Allow access from pods with client label set to "true". Ignored if `networkPolicy.allowExternal` is true.                        | `true`                   |
| `networkPolicy.extraIngress`            | Add extra ingress rules to the NetworkPolicy                                                                                     | `[]`                     |
| `networkPolicy.extraEgress`             | Add extra ingress rules to the NetworkPolicy (ignored if allowExternalEgress=true)                                               | `[]`                     |
| `networkPolicy.ingressPodMatchLabels`   | Labels to match to allow traffic from other pods. Ignored if `networkPolicy.allowExternal` is true.                              | `{}`                     |
| `networkPolicy.ingressNSMatchLabels`    | Labels to match to allow traffic from other namespaces. Ignored if `networkPolicy.allowExternal` is true.                        | `{}`                     |
| `networkPolicy.ingressNSPodMatchLabels` | Pod labels to match to allow traffic from other namespaces. Ignored if `networkPolicy.allowExternal` is true.                    | `{}`                     |
| `ingress.enabled`                       | Enable ingress record generation for neo4j                                                                                       | `false`                  |
| `ingress.pathType`                      | Ingress path type                                                                                                                | `ImplementationSpecific` |
| `ingress.apiVersion`                    | Force Ingress API version (automatically detected if not set)                                                                    | `""`                     |
| `ingress.hostname`                      | Default host for the ingress record                                                                                              | `neo4j.local`            |
| `ingress.ingressClassName`              | IngressClass that will be be used to implement the Ingress (Kubernetes 1.18+)                                                    | `""`                     |
| `ingress.path`                          | Default path for the ingress record                                                                                              | `/`                      |
| `ingress.annotations`                   | Additional annotations for the Ingress resource. To enable certificate autogeneration, place here your cert-manager annotations. | `{}`                     |
| `ingress.tls`                           | Enable TLS configuration for the host defined at `ingress.hostname` parameter                                                    | `false`                  |
| `ingress.selfSigned`                    | Create a TLS secret for this ingress record using self-signed certificates generated by Helm                                     | `false`                  |
| `ingress.extraHosts`                    | An array with additional hostname(s) to be covered with the ingress record                                                       | `[]`                     |
| `ingress.extraPaths`                    | An array with additional arbitrary paths that may need to be added to the ingress under the main host                            | `[]`                     |
| `ingress.extraTls`                      | TLS configuration for additional hostname(s) to be covered with this ingress record                                              | `[]`                     |
| `ingress.secrets`                       | Custom TLS certificates as secrets                                                                                               | `[]`                     |
| `ingress.extraRules`                    | Additional rules to be covered with this ingress record                                                                          | `[]`                     |

### Persistence Parameters

| Name                        | Description                                                                                             | Value               |
| --------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------- |
| `persistence.enabled`       | Enable persistence using Persistent Volume Claims                                                       | `true`              |
| `persistence.mountPath`     | Path to mount the volume at.                                                                            | `/bitnami/neo4j`    |
| `persistence.subPath`       | The subdirectory of the volume to mount to, useful in dev environments and one PV for multiple services | `""`                |
| `persistence.storageClass`  | Storage class of backing PVC                                                                            | `""`                |
| `persistence.annotations`   | Persistent Volume Claim annotations                                                                     | `{}`                |
| `persistence.accessModes`   | Persistent Volume Access Modes                                                                          | `["ReadWriteOnce"]` |
| `persistence.size`          | Size of data volume                                                                                     | `8Gi`               |
| `persistence.existingClaim` | The name of an existing PVC to use for persistence                                                      | `""`                |
| `persistence.selector`      | Selector to match an existing Persistent Volume for WordPress data PVC                                  | `{}`                |

### Init Container Parameters

| Name                                                        | Description                                                                                                                                                                                                                                         | Value                      |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| `volumePermissions.enabled`                                 | Enable init container that changes the owner/group of the PV mount point to `runAsUser:fsGroup`                                                                                                                                                     | `false`                    |
| `volumePermissions.image.registry`                          | OS Shell + Utility image registry                                                                                                                                                                                                                   | `REGISTRY_NAME`            |
| `volumePermissions.image.repository`                        | OS Shell + Utility image repository                                                                                                                                                                                                                 | `REPOSITORY_NAME/os-shell` |
| `volumePermissions.image.pullPolicy`                        | OS Shell + Utility image pull policy                                                                                                                                                                                                                | `IfNotPresent`             |
| `volumePermissions.image.pullSecrets`                       | OS Shell + Utility image pull secrets                                                                                                                                                                                                               | `[]`                       |
| `volumePermissions.resourcesPreset`                         | Set init container resources according to one common preset (allowed values: none, nano, small, medium, large, xlarge, 2xlarge). This is ignored if volumePermissions.resources is set (volumePermissions.resources is recommended for production). | `nano`                     |
| `volumePermissions.resources`                               | Set init container requests and limits for different resources like CPU or memory (essential for production workloads)                                                                                                                              | `{}`                       |
| `volumePermissions.containerSecurityContext.enabled`        | Enabled init container' Security Context                                                                                                                                                                                                            | `true`                     |
| `volumePermissions.containerSecurityContext.seLinuxOptions` | Set SELinux options in init container                                                                                                                                                                                                               | `{}`                       |
| `volumePermissions.containerSecurityContext.runAsUser`      | Set init container's Security Context runAsUser                                                                                                                                                                                                     | `0`                        |

### Other Parameters

| Name                                          | Description                                                      | Value   |
| --------------------------------------------- | ---------------------------------------------------------------- | ------- |
| `rbac.create`                                 | Specifies whether RBAC resources should be created               | `false` |
| `rbac.rules`                                  | Custom RBAC rules to set                                         | `[]`    |
| `serviceAccount.create`                       | Specifies whether a ServiceAccount should be created             | `true`  |
| `serviceAccount.name`                         | The name of the ServiceAccount to use.                           | `""`    |
| `serviceAccount.annotations`                  | Additional Service Account annotations (evaluated as a template) | `{}`    |
| `serviceAccount.automountServiceAccountToken` | Automount service account token for the server service account   | `true`  |

See <https://github.com/bitnami/readme-generator-for-helm> to create the table

The above parameters map to the env variables defined in [bitnami/neo4j](https://github.com/bitnami/containers/tree/main/bitnami/neo4j). For more information please refer to the [bitnami/neo4j](https://github.com/bitnami/containers/tree/main/bitnami/neo4j) image documentation.

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
helm install my-release \
  --set advertisedHost=localhost \
  --set auth.password=password123 \
    oci://REGISTRY_NAME/REPOSITORY_NAME/neo4j
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.

The above command sets the 'neo4j' native username with password `password123`. Additionally, it sets the advertised host to `localhost`.

> NOTE: Once this chart is deployed, it is not possible to change the application's access credentials, such as usernames or passwords, using Helm. To change these application credentials after deployment, delete any persistent volumes (PVs) used by the chart and re-deploy it, or use the application's built-in administrative tools if available.

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
helm install my-release -f values.yaml oci://REGISTRY_NAME/REPOSITORY_NAME/neo4j
```

> Note: You need to substitute the placeholders `REGISTRY_NAME` and `REPOSITORY_NAME` with a reference to your Helm chart registry and repository. For example, in the case of Bitnami, you need to use `REGISTRY_NAME=registry-1.docker.io` and `REPOSITORY_NAME=bitnamicharts`.
> **Tip**: You can use the default [values.yaml](https://github.com/bitnami/charts/blob/main/template/CHART_NAME/values.yaml)

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami's Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

## License

Copyright &copy; 2025 Broadcom. The term "Broadcom" refers to Broadcom Inc. and/or its subsidiaries.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.